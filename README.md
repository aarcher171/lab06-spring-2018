# ANLY502 - Massive Data Fundamentals<br>Lab L06: PIG and Hive<br>February 26, 2018

## Start your cluster

1. Start an Amazon Elastic MapReduce (EMR) Cluster using Quickstart with the following setup:
	*  Give the cluster a name that is meaningful to you
	*  Use Release `emr-5.12.0`
	*  Select the first option under Applications (Core Hadoop: Hadoop 2.8.3 with Ganglia 3.7.2, Hive 2.3.2, Hue 4.1.0, Mahout 0.13.0, Pig 0.17.0, and Tez 0.8.4)
	*  Select 1 master and 2 core nodes, using `m4.large` instance types
	*  Select your correct EC2 keypair or you will not be able to connect to the cluster
	*  Click **Create Cluster**

2. Once the cluster is up and running and in "waiting" state, ssh into the master node: `ssh hadoop@[[master-node-dns-name]]`

3. Install git on the master node: `sudo yum install -y git`

3. Clone this repository to the master node. Note: since this is a public repository you do can use the `http` GitHub URL: `git clone https://github.com/gu-anly502/lab06-spring-2018.git`

4. Change directory into the lab: `cd lab06-spring-2018` 

## PIG Exercises

1. Look at the contents of the file `pigdemo.tx`

	```
	[hadoop@ip-172-31-2-208 lab06-spring-2018]$ cat pigdemo.txt
	SD	Rich
	NV	Barry
	CO	George
	CA	Ulf
	IL	Danielle
	OH	Tom
	CA	manish
	CA	Brian
	CO	Mark
	```

2. Start the Grunt Shell: `pig`

3. You can run HDFS commands from the Grunt Shell:
	- `grunt> ls`
	- Make a directory within the cluster HDFS called lab06: `mkdir lab06`
	- Copy the `pigdemo.txt` file from the **local** filesystem to HDFS: `copyFromLocal pigdemo.txt lab06/`
	- Check to make sure the file was copied: 
	
	```
	grunt> ls lab06/
	hdfs://ip-172-31-2-208.ec2.internal:8020/user/hadoop/lab06/pigdemo.txt<r 2>	80
	```
	
1. Define the employees relation, using a schema:

	```
	grunt> employees = LOAD 'lab06/pigdemo.txt' AS (state, name);
	```

4. Use the describe command to see what a relation looks like:
	
	```
	grunt> describe employees;
	employees: {state: bytearray,name: bytearray}
	```

4. View the records in the employees relation:

	```
	grunt> DUMP employees
	... lots of text from the spawned MapReduce job ...
	(SD,Rich)
	(NV,Barry)
	(CO,George)
	(CA,Ulf)
	(IL,Danielle)
	(OH,Tom)
	(CA,manish)
	(CA,Brian)
	(CO,Mark)
	```
	
4. Filter the relation by a field

	```
	grunt> ca_only = FILTER employees BY (state=='CA');
	681844 [main] WARN  org.apache.pig.newplan.BaseOperatorPlan  - Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	18/02/26 17:23:01 WARN newplan.BaseOperatorPlan: Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	grunt>
	```

4. See the contents of the filtered relation. The output is still tuples but only the records that match the filter.

	```
	grunt> DUMP ca_only
	... lots of text from the spawned MapReduce job ...
	(CA,Ulf)
	(CA,manish)
	(CA,Brian)
	```
	
4. Create a group

	```
	grunt> emp_group = GROUP employees BY state;
	1186375 [main] WARN  org.apache.pig.newplan.BaseOperatorPlan  - Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	18/02/26 17:31:25 WARN newplan.BaseOperatorPlan: Encountered Warning IMPLICIT_CAST_TO_CHARARRAY 1 time(s).
	```


4. Describe the new relation. Bags represent groups in PIG. A Bag is an unordered collection of tuples;

	```
	grunt> describe emp_group
	emp_group: {group: bytearray,employees: {(state: bytearray,name: bytearray)}}
	```

4. See the contents of the group:

	```
	grunt> DUMP emp_group
	... lots of text from the spawned MapReduce job ...
	(CA,{(CA,Ulf),(CA,manish),(CA,Brian)})
	(CO,{(CO,George),(CO,Mark)})
	(IL,{(IL,Danielle)})
	(NV,{(NV,Barry)})
	(OH,{(OH,Tom)})
	(SD,{(SD,Rich)})
	```
4. Use the `STORE` command to write a Relation back to HDFS (or S3). Notice that the name given is that of a directory, not of a file

	```
	grunt> STORE emp_group INTO 'emp_group';
	... lots of text from the spawned MapReduce job ...
	Input(s):
	Successfully read 9 records (80 bytes) from: "hdfs://ip-172-31-2-208.ec2.internal:8020/user/hadoop/lab06/pigdemo.txt"
	
	Output(s):
	Successfully stored 6 records (128 bytes) in: "hdfs://ip-172-31-2-208.ec2.internal:8020/user/hadoop/emp_group"
	```


## Hive Exercises
