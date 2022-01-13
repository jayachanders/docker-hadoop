# Credit
Taken from the following github projects
https://github.com/big-data-europe/docker-hadoop
https://github.com/big-data-europe/docker-spark
https://github.com/big-data-europe/docker-hadoop-spark-workbench

Actual testing is done in another project
https://github.com/jayachanders/docker-hadoop-spark

# Changes

Version 2.0.0 introduces uses wait_for_it script for the cluster startup

# Hadoop Docker

## Quick Start

To deploy an example HDFS cluster, run:
```
  docker-compose up
```

`docker-compose` creates a docker network that can be found by running `docker network list`, e.g. `dockerhadoop_default`.

Run `docker network inspect` on the network (e.g. `dockerhadoop_default`) to find the IP the hadoop interfaces are published on. Access these interfaces with the following URLs:

* Namenode: http://<dockerhadoop_IP_address>:9870/dfshealth.html#tab-overview
* History server: http://<dockerhadoop_IP_address>:8188/applicationhistory
* Datanode: http://<dockerhadoop_IP_address>:9864/
* Nodemanager: http://<dockerhadoop_IP_address>:8042/node
* Resource manager: http://<dockerhadoop_IP_address>:8088/
* Spark master: http://<dockerhadoop_IP_address>:8080/
* Spark worker: http://<dockerhadoop_IP_address>:8081/
* Hue (HDFS Filebrowser): http://<dockerhadoop_IP_address>:8088/home

## Connect to namenode , datanode, or other dockers
Go to the command line of the Spark master and start PySpark.
```
  docker exec -it namenode bash
```

## Create a HDFS directory /dis_materials and Copy data to hdfs 
```
# Copies data
  docker cp dis_materials namdenode:/
# Connect to namenode
  docker exec -it namenode bash
# Create directory
  hdfs dfs -mkdir -p /dis_materials
# Copy data from namenode to hdfs
  hdfs dfs -put dis_materials/*.txt dis_materials/*.csv /dis_materials
```

## Analyzing unstructured data with Hadoop Streaming from namenode only
```
cd /dis_materials

hadoop jar /opt/hadoop-3.2.1/share/hadoop/tools/lib/hadoop-streaming-3.2.1.jar \
    -mapper email_count_mapper.py \
    -reducer email_count_reducer.py \
    -input /dis_materials/hadoop_1m.txt \
    -output /dis_materials/output1 \
    -file email_count_mapper.py \
    -file email_count_reducer.py

hadoop fs -text /dis_materials/output1/part*
```

## Analysing unstructured data with MRJob
```
python3 count_sum.py --hadoop-streaming-jar /opt/hadoop-3.2.1/share/hadoop/tools/lib/hadoop-streaming-3.2.1.jar -r hadoop hdfs:///dis_materials/hadoop_1m.txt --output-dir hdfs:///dis_materials/output2 --no-output

hadoop fs -text /dis_materials/output2/part* | less
```

#Testing code without any Hadoop installation
```
cat hadoop_1m.txt | ./email_count_mapper.py | sort -k1,1 | ./email_count_reducer.py
python count_sum.py -r inline hadoop_1m.txt
```

## Quick Start Spark (PySpark)

Go to http://<dockerhadoop_IP_address>:8080 or http://localhost:8080/ on your Docker host (laptop) to see the status of the Spark master.

Go to the command line of the Spark master and start PySpark.
```
  docker exec -it spark-master bash

  /spark/bin/pyspark --master spark://spark-master:7077
```
Once connected to the pyspark, follow the commands associated with the spark.py

# read data from hdfs in Spark:
```
df1 = spark.read.csv("hdfs://namenode:9000/dis_materials/hadoop_1m.csv")
df1.show()

df1 = spark.read.txt("hdfs://namenode:9000/dis_materials/hadoop_1m.txt")
df1.show()

```

# Stop all dockers
```
docker stop $(docker ps -q)
```

# Stop all dockers with bde2020 images only
```
docker ps -a | grep bde2020 | awk '{print $1}' | xargs docker stop
docker stop $(docker ps -a | grep bde2020 | awk '{print $1}' )
```

# Start all stopped dockers
```
docker start $(docker ps -aq)
```

# Start all dockers with bde2020 images only
```
docker ps -a | grep bde2020 | awk '{print $1}' | xargs docker start
docker start $(docker ps -a | grep bde2020 | awk '{print $1}' )
```


# Delete docker volumes:
```
docker rmi $(docker images 'bde2020/*')
docker volume rm $(docker volume ls | grep 'docker-hadoop')
```
