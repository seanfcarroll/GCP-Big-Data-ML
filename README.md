# GCP-Big-Data-ML
Notes for the Google Cloud Platform Big Data and Machine Learning Fundamentals course.

https://www.coursera.org/learn/gcp-big-data-ml-fundamentals/home/welcome

https://console.cloud.google.com

## 1. Google Cloud Architecture

    -----------------------------------------
            2. Big Data/ML Products
    -----------------------------------------
     1. Compute Power | Storage | Networking
    -----------------------------------------
               0. Security
    -----------------------------------------


-2. Abstract away scaling, infrastructure

-1. Process, store, and deliver pipelines, ML models, etc.

-0. Auth

## 2. Creating a VM on Compute Engine

### 2.1 Setting up the VM

1. https://cloud.google.com
2. Compute --> Compute Engine --> VM Instances
3. Create a new VM

3.1 Allow full access to cloud APIs

3.2 Since we will access the VM through SSH, we don't need to allow HTTP or HTTPS traffic

4. Click SSH to connect

At this stage, the new VM has no software.

We can just install stuff like normal:

`sudo apt-get install git`

(APT: advanced package tool, package manager)

### 2.2 Sample earthquake data

After installing git, we can pull down the data from our repo:

`git clone https://www.github.com/GoogleCloudPlatform/training-data-analyst`

Course materials are in:

`training-data-analyst/courses/bdml_fundamentals`

Go to the Earthquake sample in `earthquakevm`.

The `ingest.sh` shell script contains the script to ingest data (`less ingest.sh`). This script basically just deletes existing data and then `wget`s a new CSV file containing the data.

Now, `transform.py` contains a Python script to parse the CSV file and create a PNG from it using `matplotlib` (
[matplotlib notes](https://nbviewer.jupyter.org/github/pekoto/MyJupyterNotes/blob/master/Python%20for%20Data%20Analysis.ipynb)).

([File details](https://github.com/GoogleCloudPlatform/datalab-samples/blob/master/basemap/earthquakes.ipynb))

Run `./install_missing.sh` to get the missing Python libraries, and run the scripts mentioned above to get the CSV and generate the image.

### 2.3 Transferring the data to bucket storage

Now, since we have generated the image. Let's get it off the VM and delete the VM.

To do this, we need to create some storage:
Storage > Storage > Browser > Create Bucket

Now, to view our bucket, we can use:

`gsutil ls gs://[bucketname]` (hence why bucket names need to be globally unique)

To copy out data to the bucket, we can use:

`gsutil cp earthquakes.* gs://[bucketname]`

### 2.4 Stopping/deleting the VM

So now we're finished with our VM, we can either:

__STOP__
Stop the machine. You will still pay for the disk, but not the processing power.

__DELETE__
Delete the machine. You won't pay for anything, but obviously you will lose all of the data.

### 2.5 Viewing the assets

Now, we want to make our assets in storage publically available.
To do this:
Select Files > Permissions > Add Members > `allUsers` > Grant `Storage Object Viewer` role.

Now, you can use the public link to [view your assets](https://storage.googleapis.com/earthquakeg/earthquakes.htm).

## 3. Storage

There are 4 storage types:

1. __Multiregional__: Optimized for geo-redundancy, end-user latency
2. __Regional__: High performance local access (common for data pipelines you want to run, rather than give global access)
3. __Nearline__: Data accessed less than once a month
4. __Coldline__: Data accessed less than once a year

## 4. Edge Network (networking)

Edge node receives the user's request and passes to the nearest Google data center.

Consider node types (Hadoop):

1. Master node: Controls which nodes perform which tasks. Most work is assigned to...
2. Worker node: Stores data and performs calculations
3. Edge node: Facilitate communication between users and master/worker nodes

__Edge computing__: brings data storage closer to the location where it's needed. In contrast to cloud computing, edge computing does decentralized computing at the edge of the network.

__Edge Node (aka. Google Global Cache - GGC)__: Points close to the user. Network operators and ISPs deploy Google servers inside their network. E.g., YouTube videos could be cached on these edge nodes.

__Edge Point of Prescence__: Locations where Google connects its network to the rest of the internet.

https://peering.google.com/#/

## 5. Security

Use Google IAM, etc. to provide security.

BigQuery data is encrypted.

## 6. Big Data Tool History

1. __GFS__: Data can be stored in a distributed fashion
2. __MapReduce__: Distributed processing, large scale processing across server clusters. Hadoop implementation
3. __BigTable__: Record high volume of data
4. __Dremel__: Breaks data into small chunks (shards) and compresses into columnar format, then uses query optimizer to process queries in parallel (service auto-manages data imbalances and scale)
5. __Colossus, TensorFlow__: And more...

## (Note on Columnar Databases)

Consider typical row-oriented databases. The data is stored in a row. This is great when you want to query multiple columns from a single row.

However, what if you want to get all of the data from all rows from a single column?

For this, we need to read every row, picking out just the columns we want. For example, we want the average age of all the people. This will be slower in row-oriented databases, because even an index on the age column wouldn't help. The DB will just do a sequential scan.

Instead we can store the data column-wise. How does it know which columns to join together into a single set? Each column has a link back to the row number. Or, in terms of implementation, each column could be stored in a separate file. Then, each column for a given row is stored at the same offset in a given file. When you scan a column and find the data you want, the rest of the data for that record will be at the same offset in the other files.


__Row oriented__
```
RowId 	EmpId 	Lastname 	Firstname 	Salary
001 	10 	    Smith 	    Joe      	40000
002 	12 	    Jones 	    Mary 	    50000
003 	11 	    Johnson 	Cathy 	    44000
004 	22 	    Jones 	    Bob 	    55000 
```

__Column oriented__
```
10:001,12:002,11:003,22:004;
Smith:001,Jones:002,Johnson:003,Jones:004;
Joe:001,Mary:002,Cathy:003,Bob:004;
40000:001,50000:002,44000:003,55000:004;
```

In terms of IO improvements, if you have 100 rows with 100 columns, it will be the difference between reading 100x2 vs. 100x100.

It also becomes easier to horizontally scale -- make a new file to store more of that column.

It is also easier to add a column -- just add a new file.

