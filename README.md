# hadoop-yarn-spark-docker-cluster
This repository contains Docker images for Apache Spark executed on Hadoop YARN.

Upgraded project downloaded from https://github.com/bartosz25/spark-docker in order to be compatible with new versions of Spark/Hadoop and whether use on **Windows 10 Docker** or **Linux**.

**Image versions: Ubuntu 20.04, Spark v3.3.2 and Hadoop v3.3.2**

The purpose of them is to allow programmers to test Spark applications deployed on YARN easier. 
**It was not designed to be deployed in production environments**.

# Building the cluster

Download and unzip all files inside a folder named "hadoop-spark-cluster".

This project uses `docker-compose` to create a master and worker containers (nodes). It's executed with standard `docker-compose up` command and the number of workers is  defined with `--scale slave=X` property.

But before calling it, 3 Docker images must be built executing this windows powershell script:
```
build_images.ps1
```
On Linux:
```
make build_base_image
make build_master_image
make build_slave_image
```
Now we can create the cluster with a master and 3 slaves:

NOTE: one point of attention is that the more slaves you put, the greater the amount of memory available in the cluster. By default, the cluster adds 8GB for each slave. As my PC has 32GB, I created a cluster with 2 containers that together use a total of 16GB. For my tests this is enough, but nothing prevents you from changing the settings to suit your needs if you wish)
```
docker-compose up -d --scale slave=3
``` 
Stops containers and removes containers, networks, volumes, and images created by up:
```
docker-compose down
``` 
If the cluster exists and you just want to start or stop it:
```
docker-compose start
``` 
```
docker-compose stop
``` 

# Spark and Hadoop WebUIs
Spark, HDFS and YARN expose web UI used to track the execution of the applications:
* http://localhost:8088  - YARN UI's address
* http://localhost:18080 - Spark history UI's address
* http://localhost:50070 - Hadoop HDFS NodeManager WebUI

# Repository structure: Volumen linked
* conf-master: stores master's configuration files are stored there
* conf-slave: stores slave's configuration files are stored there 
* master: contains master's Dockerfile
* slave: contains slave's Dockerfile
* shared-master: this repository is shared between master's Docker container (/home/sparker/shared) and host. 
* shared-slave: this repository is shared between slave Docker containers (/home/sparker/shared) and host

Shared repositories can be used to, for example, put the JAR executed with spark-submit inside.

# To get access and run spark-submit commands inside the cluster
Get the container Id of master node.
```
docker ps
```
Type the following in a terminal to access container terminal:
```
docker exec -it #container_id /bin/bash
``` 
If you have problems with docker container names, check it
```
docker ps --format '{{.Names}}'
``` 

# Tests
To verify that the cluster was correctly installed, launch _SparkPi_ example:
```
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --driver-memory 512m --executor-memory 512m --executor-cores 1 ~/spark-2.4.8-bin-hadoop2.7/examples/jars/spark-examples*.jar 10000
```

# Troubleshooting
## "Unhealthy Node local-dirs and log-dirs"
I encounter the issue when I had too few available disk space. It makes that the slave nodes are detected as unhealthy. You can fix that either by playing with the configuration 
https://stackoverflow.com/questions/29131449/why-does-hadoop-report-unhealthy-node-local-dirs-and-log-dirs-are-bad or simply by ensuring that you have enough free disk space.


