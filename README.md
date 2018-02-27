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
	
4. To see all the relations that you have created: `aliases`


## Use PIG to generate a dataset for Hive

1. Exit the Grunt Shell if you haven't already: `grunt> quit;`
2. Make sure you are within the lab directory. You can check by using the Linux command `pwd` (present working directory). You should be in `/home/hadoop/lab06-spring-2018`, if not change to it.
3. Unzip the White House visits dataset: `unzip whitehouse_visits.zip`. This creates a file called `whitehouse_visits.txt`
4. Create a directory within HDFS: `hadoop fs -mkdir whitehouse`
5. Copy the `whitehouse_visits.txt` file from the local filesystem into HDFS
	`hadoop fs -put whitehouse_visits.txt whitehouse/visits.txt` (Note the file has been renamed within HDFS)
5. Explore the contents of the Pig file `wh_visits.pig`. This is a pre-written Pig script that loads the `visits.txt` and extracts certain fields, filters, and writes the output to a location in HDFS: `cat wh_visits.pig`
6. Run the Pig script: `pig wh_visits.pig`
7. You should have a set of new files in the `hive_wh_visits` directory inside HDFS:

	```
	[hadoop@ip-172-31-18-99 lab06-spring-2018]$ hadoop fs -ls
	Found 3 items
	drwxr-xr-x   - hadoop hadoop          0 2018-02-26 20:29 hive_wh_visits
	drwxr-xr-x   - hadoop hadoop          0 2018-02-26 18:14 lab06
	drwxr-xr-x   - hadoop hadoop          0 2018-02-26 20:28 whitehouse
	[hadoop@ip-172-31-18-99 lab06-spring-2018]$ hadoop fs -ls hive_wh_visits
	Found 3 items
	-rw-r--r--   1 hadoop hadoop          0 2018-02-26 20:29 hive_wh_visits/_SUCCESS
	-rw-r--r--   1 hadoop hadoop     971339 2018-02-26 20:29 hive_wh_visits/part-v000-o000-r-00000
	-rw-r--r--   1 hadoop hadoop     142850 2018-02-26 20:28 hive_wh_visits/part-v000-o000-r-00001
	```
8. Explore the contents of the results file/files:

	```
	[hadoop@ip-172-31-18-99 lab06-spring-2018]$ hadoop fs -cat hive_wh_visits/part-v000-o000-r-00000 | head
	BUCKLEY	SUMMER	10/12/2010 14:48	10/12/2010 14:45	WH
	CLOONEY	GEORGE	10/12/2010 14:47	10/12/2010 14:45	WH
	PRENDERGAST	JOHN	10/12/2010 14:48	10/12/2010 14:45	WH
	LANIER	JAZMIN		10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	ELIZABETH	10/13/2010 12:34	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	GREGORY	10/13/2010 12:35	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	JOANNE	10/13/2010 12:35	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	KATHERINE	10/13/2010 12:34	10/13/2010 13:00	WH	BILL SIGNING/
	MAYNARD	PHILIP	10/13/2010 12:35	10/13/2010 13:00	WH	BILL SIGNING/
	MOHAN	EDWARD	10/13/2010 12:37	10/13/2010 13:00	WH	BILL SIGNING/
	cat: Unable to write to output stream.
	```
8. You will use this file as an input to Hive in the next exercise
	
## Hive Exercises

1. Start the hive console:

	```
	[hadoop@ip-172-31-18-99 lab06-spring-2018]$ hive
	
	Logging initialized using configuration in file:/etc/hive/conf.dist/hive-log4j2.properties Async: false
	hive>
	```
	
2. Create an **External Table** from the file created earlier:

	```
	create external table wh_visits (
	lname string, 
	fname string,
	time_of_arrival string, 
	appt_scheduled_time string,
	meeting_location string,
	info_comment string
	) 
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY '\t' 
	LOCATION  '/user/hadoop/hive_wh_visits/';
	```

3. Count the number of records in the `wh_visits` table:

	```
	select count(*) from wh_visits;
	```

4. Explore 20 records: 

	```
	select * from wh_visits limit 20;
	```

5. Write a SQL query that gives you the count of comments for records where the comment is not empty.

6. Once you figure out the correct SQL statement, create a CSV file in HDFS with the results of the query from the previous step. The syntax to use is the following:

	```
	INSERT OVERWRITE DIRECTORY '[[hdfs/s3 output directory]]'
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ',' 
	select ...;
	```
	Where:
	- `[[hdfs/s3 output directory]]` is the name of a **directory** to be created, where the results files from the MapReduce process will be placed.
	- `select ...` is the query




