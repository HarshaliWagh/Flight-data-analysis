# [Flights On Time](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/HG7NV7)

The three primary tasks perfomed by me in this project were to Design, implement, and run an Oozie workflow to find out
- The 3 airlines with the highest and lowest probability, respectively, for being on schedule;
 - The 3 airports with the longest and shortest average taxi time per flight (both in and out), respectively
 - The most common reason for flight cancellations.


# Requirements

1. My workflow contains three MapReduce jobs that run in fully distributed mode.
2. I have run the workflow to analyze the entire data set (total 22 years from 1987 to 2008) at one time on two VMs first and then gradually increase the system scale to the maximum allowed number of VMs for at least 5 increment steps, and measure each corresponding workflow execution time.
3. In order to analyze workflow of the data in a progressive manner with an increment of 1 year, i.e. the first year (1987), the first 2 years (1987-1988), the first 3 years (1987-1989), …, and the total 22 years (1987-2008), on the maximum allowed number of VMs, and measured each corresponding workflow execution time.


# Project Report

### a) Structure of Oozie Workflow
![](https://github.com/Gonnuru/flight_data_analysis/blob/master/images/workflow.png)

### b) Algorithms Used

***First Map-Reduce: On Schedule Airlines***
1. Mapper <key,value>:<UniqueCarrier,1 or 0>
2. The Mapper read the data line by line, ignore the first line and the NA data. If the data
of the ArrDelay column which is less than or equal to 10 minutes, output:
<UniqueCarrier,1>, otherwise output: <UniqueCarrier,0>
3. Reducer <key,value>:<UniqueCarrier,probability>
Probability = (# of 1) / (# of 1 and 0)
4. Reducer sum the values from the mapper of the same key, the sum will be the number of
this airline when it is on schedule. And calculate the total number of 0 and 1, then
calculate the on schedule probability of this airline.
5. Reducer then use the Comparator function do the sorting. After sorting, output the 3
airlines with the highest and lowest probability.
6. If the data is NULL, then output: There is no value can be used, so no output.

***Second Map-Reduce: Airports Taxi Time***
1. Mapper <key,value>: <IATA airport code, TaxiTime>: <Origin,TaxiOut> or <Dest,TaxiIn>
2. The Mapper read the data line by line, ignore the first line. If the data of the TaxiIn or the
TaxiOut column is not NA, output: <IATA airport code, TaxiTime>
3. Reducer <key,value>: <IATA airport code, Average TaxiTime>
4. Reducer sum the value from the mapper of the same key (normal), and calculate the total
times this key is found (all). Then do the equation: normal/all to calculath the average
TaxiTime of each key.
5. Reducer then use the Comparator function do the sorting. After sorting, output the 3
airports with the longest and shortest average taxi time.
6. If the data is NULL, then output: There is no value can be used, so no output.

***Third Map-Reduce: Cancellation Reasons***
1. Mapper <key,value>: < CancellationCode, 1>
2. The Mapper read the data line by line, ignore the first line. If the value of the Cancelled is
1 and the CancellationCode is not NA, output: < CancellationCode, 1>
3. Reducer <key,value>: < CancellationCode, sum of the 1s>
4. Reducer sum the value from the mapper of the same key.
5. Reducer then use the Comparator function do the sorting. After sorting, output the most
common reason for flight cancellations.
6. If the data is NULL, then output: There is no the most common reason for flight
cancellations.

### C) Performance Measurement Plot for Increasing VM's (Entire Dataset)![](https://github.com/Gonnuru/flight_data_analysis/blob/master/images/vms.png)

According to the Figure that compares the workflow execution time in
response to an increasing number of VMs used for processing the entire data set (22
years), along ***with the increasing the number of the VMs, the workflow execution time will decrease***. By increasing the number of the VMs, the processing ability of the hadoop cluster will also increase, because the data can be dealed with in parallel on more datanodes. Then the execution time of every map-reduce job will be shorter than before, thus the execution time of the oozie workflow will be shorter than before too. However, the execution time of deal with the same data size will not always decrease by increasing the number of VMs. When the execution time decrease to a certain range, although trying to increase the number of VM, the execution time will no longer decreasing anymore. The reason is more VMs means more information interaction time between the datanodes of a hadoop cluster. Information interaction time of a hadoop cluster increases when the number of VMs increases.

### d) Performance Measurement Plot Increasing data Size (20 VMs)![](https://raw.githubusercontent.com/Gonnuru/flight_data_analysis/master/images/decreasing%20vms.png)
According to the Figure that compares the workflow execution time in response to an increasing data size (from 1 year to 22 years), ***the execution time of the oozie workflow will always increase too with increasing data size***. In the beginning, the time-consuming increase with the increase in the amount of data, but the time-consuming increasing is slow, this is because the data increasing of first few years is not that much. On the contrary, after year 1998, the time-consuming increase very fast, the slope is become much steep compare to the first few years. The reason is the flight data between year 1998 to year 2008 is increase faster than the previous years. It also shows more and more people choose traveling by plane.

### Some Environment setting of my Hadoop cluster

Instance Information: Ubuntu Server 16.04 LTS (HVM)
Family: General purpose
Type: t2.medium
vCPUs: 2
Memory(GiB): 6
Instance Storage(GB): 30GB
Environment of master instance and slave instances:
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export PATH=${JAVA_HOME}/bin:${PATH}
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:/usr/local/hadoop/bin
export OOZIE_HOME=/usr/local/oozie/distro/target/oozie-4.3.0-distro/oozie-4.3.0
export PATH=$PATH:$OOZIE_HOME/bin
Local: Windows Operation System
User use MobaXterm to manage instances and files





