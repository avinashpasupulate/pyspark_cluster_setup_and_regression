FROM debian:latest
RUN apt-get update && \
    apt-get install --no-install-recommends -y python3.5 \
    curl \
    default-jre \
    scala && \
    rm -rf /var/lib/apt/lists* && \
    curl https://bootstrap.pypa.io/ez_setup.py -o - | python3.5 && \
    easy_install pip && \
    pip uninstall tornado && \
    pip install tornado==5.1.1 \
    py4j==0.10.7 \
    jupyter \
    pyspark

#setting up spark
RUN rm /bin/sh && ln -s /bin/bash /bin/sh && \
    curl -O http://archive.apache.org/dist/spark/spark-2.0.0/spark-2.0.0-bin-hadoop2.7.tgz && \
    tar -zxvf spark-2.0.0-bin-hadoop2.7.tgz && \
    mv spark-2.0.0-bin-hadoop2.7 /spark && \
    rm spark-2.0.0-bin-hadoop2.7.tgz


#setting up environment variables
ENV SPARK_HOME='/spark'
ENV PATH=$SPARK_HOME:$PATH
ENV PYTHONPATH=$SPARK_HOME/python:$PYTHONPATH
ENV JAVA_HOME='/usr/lib/jvm/java-8-openjdk-amd64'


#sourcing the environment variables
RUN source ~/.bashrc && \
    #setting working directory
    mkdir /home/jupyter && \
    mkdir /home/jupyter/notebooks
WORKDIR /home/jupyter/

RUN apt-get clean

#Running jupyter by default
CMD ["sh","-c","jupyter notebook --ip 0.0.0.0 --allow-root --no-browser"]


