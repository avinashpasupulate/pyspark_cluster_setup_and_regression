
Distributed Analysis Using Apache Spark Standalone Cluster Deployed On Docker Containers

<<editing in progress>>
  
<<make sure images are not copyright protected and proper citing is done>>

Refer https://blog.medium.com/best-practices-for-writing-on-medium-386506ae62b9


In this article I will be describing one of the several quests for knowledge in my journey towards exploring distributed methods for data analysis.

Two frameworks / tools that have always intrigued me were Apache Spark and Docker with their inexhaustible opportunities to be applied in the data science pipelines




Apache Spark 
Apache is a framework for performing distributed data analysis across several nodes in a cluster, managed by a master node.

Applications in Data science


Docker
Docker which is widely popular in the application development community aiding towards creating CI/CD pipelines and deploying applications across several containers on the server node.

Applications in Data science

Here I would be listing  my steps take in launching a spark cluster on three docker containers and running a linear regression model on the spark cluster.

Some basic tips to create the Dockerfile of lower size,
	•	It is good to use a smaller base image to reduce the overhead
	•	Remove / purge downloaded packages when done installing them
	•	Minimize the number of layers 

**Make sure that the versions of packages and dependencies used in the containers are the same across all jupyter, master and worker nodes. Also ensure that the environmental variables are set properly to prevent issues with running pyspark on jupyter**




Not checking on the above made me spend several hours trying to figure out what was wrong, should have read the documentation :(

Creating a Dockerfile with the configuration for the cluster

FROM debian:latest

We would be using  debian as a base for building our spark image

RUN apt-get update && \     apt-get install --no-install-recommends -y python3.5 \     curl \     default-jre \     scala && \     rm -rf /var/lib/apt/lists* && \     curl https://bootstrap.pypa.io/ez_setup.py -o - | python3.5 && \     easy_install pip && \     pip install py4j==0.10.7 \     jupyter \     pyspark
The following lines would install the dependencies and the tools required to set them up. And purge the packages when done.

RUN apt-get update && \     apt-get install --no-install-recommends -y python3.5 \     curl \     default-jre \     scala && \     rm -rf /var/lib/apt/lists* && \


Installing the required python packages, remember pyspark depends on p4j to transfer the tasks to the underlying jvm. Currently only py4j version 0.10.7 is supported by pyspark.

Also installing jupyter to interact with the spark cluster through pyspark.

    curl https://bootstrap.pypa.io/ez_setup.py -o - | python3.5 && \     easy_install pip && \     pip install py4j==0.10.7 \     jupyter \     pyspark


Using the below lines we would be setting up spark and hadoop 

#setting up spark RUN rm /bin/sh && ln -s /bin/bash /bin/sh && \     curl -O http://archive.apache.org/dist/spark/spark-2.0.0/spark-2.0.0-bin-hadoop2.7.tgz && \     tar -zxvf spark-2.0.0-bin-hadoop2.7.tgz && \     mv spark-2.0.0-bin-hadoop2.7 /spark && \     rm spark-2.0.0-bin-hadoop2.7.tgz && \     #installing hadoop     curl -O http://archive.apache.org/dist/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz && \     tar -zxvf hadoop-2.8.0.tar.gz && \     mv hadoop-2.8.0 /hadoop && \     rm hadoop-2.8.0.tar.gz

Setting up the environment variables to create a reference point for the installed dependencies

#setting up environment variables ENV SPARK_HOME='/spark' ENV PATH=$SPARK_HOME:$PATH ENV PYTHONPATH=$SPARK_HOME/python:$PYTHONPATH ENV HADOOP_HOME='/hadoop' ENV HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native ENV HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib" ENV JAVA_HOME='/usr/lib/jvm/java-8-openjdk-amd64'

#sourcing the environment variables RUN source ~/.bashrc && \     #setting working directory     mkdir /home/jupyter && \     mkdir /home/jupyter/notebooks WORKDIR /home/jupyter/


Sourcing the environment variables into bashrc file to initialize the reference points
#sourcing the environment variables RUN source ~/.bashrc && \


Clearing the downloaded packages from apt-get install
RUN apt-get clean


Running jupyter by default when the container is started, although /bin/sh can be used to run the shell when starting the container to run spark-master and spark-worker nodes 
 
<<remove below line>>
The below line could be excluded and can be run once the container is started

#Running jupyter by default CMD ["sh","-c","jupyter notebook --ip 0.0.0.0 --allow-root --no-browser"]
Building the Image with a tag of avipasup (username) / jupyter-spark (image name) : latest (image tag) don’t forget the . at the end

user$ docker build -t avipasup/jupyter-spark:latest .
 

Launch Containers

Before launching the containers create a docker network to launch the clusters into a local network.


Jupyter Container 

Once the image is built, it can be run to create a container with port mapping

user$ docker run --rm -it -p 8888:8888 --network spark_network avipasup/jupyter-spark:latest

The above line would launch the container and run jupyter notebook by default, use the resulting url to open the jupyter notebook environment.

The below line can be append, launch directly into the shell

/bin/sh


Spark Master

Launching the image into container directly into a shell
user$ docker run --rm -it -p 8080:8080 -p 7077:7077 --name spark-master -hostname spark-master --network spark_network avipasup/jupyter-spark:latest /bin/sh


Run the code from the root directory where the ‘/spark” folder is present , note the `` around hostname
sh.# /spark/bin/spark-class org.apache.spark.deploy.master.Master --ip `hostname` --webui-port 8080 --port 7077 
Spark Worker

Launching the image into container directly into a shell
user$ docker run --rm -it -p 8080:8080 --name spark-worker01 -hostname spark-worker01 --network spark_network avipasup/jupyter-spark:latest /bin/sh


Run the code from the root directory where the ‘/spark” folder is present, mention the spark-master ip to register the spark-worker

sh.# /spark/bin/spark-class org.apache.spark.deploy.master.Master --webui-port 8080 spark://spark-master:7077 

Multiple spark-workers can be launched using the above code, just remember to change the hostname and name to a more suitable name eg. spark-worker02 when launching the container.

Once the clusters are launched you will find them registered on the spark-master console or at http://localhost:8080 



Jupyter Notebooks

Create a new notebook and run the below python code to create a spark session,


[1]: from pyspark.sql import SparkSession

[2]: spark = SparkSession                          .builder\                          .appName('pyspark_app')\                          .master('spark://spark-master:7077')\                          .config('spark.submit.deployMode','cluster')\                          .getOrCreate()


If the containers have been setup properly there should be no resulting errors, also keep an eye on the container console for more specific error messages. And the application name (in this case ‘pyspark_app’) would appear on the master ip http://localhost:8080 
Now that we have created a spark session let us import other methods from the pyspark’s mllib

** remember to restart the kernel and clear the output before running the whole notebook again, and also kill unfinished applications from the spark-master web ui**


Importing other required methods,

 [3]: from pyspark.ml.feature import VectorAssembler

 [4]: from pyspark.ml.regression import LinearRegression



Further Steps to Explore

	•	Setting up NFS and HDFS to be used for imported data
	•	Use docker swarm to auto scale containers for High Availability solutions
	•	Deploy containers and run analytics tasks on a cluster of SBCs (Odroid XU4s and a Raspberry Pi)


Reference:

	•	https://spark.apache.org/docs/latest/

	•	https://towardsdatascience.com/a-journey-into-big-data-with-apache-spark-part-1-5dfcc2bccdd2

	•	https://blog.medium.com/best-practices-for-writing-on-medium-386506ae62b9

	•	
