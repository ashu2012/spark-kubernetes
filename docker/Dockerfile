FROM python:3.7-slim-stretch

# PATH
ENV PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Spark
ENV SPARK_VERSION 3.0.0
ENV SPARK_HOME /usr/local/spark
ENV SPARK_LOG_DIR /var/log/spark
ENV SPARK_PID_DIR /var/run/spark
ENV PYSPARK_PYTHON /usr/local/bin/python
ENV PYSPARK_DRIVER_PYTHON /usr/local/bin/python
ENV PYTHONUNBUFFERED 1
ENV HADOOP_COMMON org.apache.hadoop:hadoop-common:2.7.7
ENV HADOOP_AWS org.apache.hadoop:hadoop-aws:2.7.7
ENV SPARK_MASTER_HOST sparkmaster

# Java
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/

# Install curl
RUN apt-get update && apt-get install -y curl

# Install procps
RUN apt-get install -y procps

# Install coreutils
RUN apt-get install -y coreutils

# https://github.com/geerlingguy/ansible-role-java/issues/64
RUN apt-get update && mkdir -p /usr/share/man/man1 && apt-get install -y openjdk-8-jdk && \
    apt-get install -y ant && apt-get clean && rm -rf /var/lib/apt/lists/ && \
    rm -rf /var/cache/oracle-jdk8-installer;

# Download Spark, enables full functionality for spark-submit against docker container
RUN curl http://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop2.7.tgz | \
        tar -zx -C /usr/local/ && \
        ln -s spark-${SPARK_VERSION}-bin-hadoop2.7 ${SPARK_HOME}

# add scripts and update spark default config
ADD spark-master  /spark-master.sh
ADD spark-worker  /spark-worker.sh
ADD common.sh  /common.sh

RUN chmod +x /common.sh /spark-master.sh /spark-worker.sh

ADD spark-defaults.conf ${SPARK_HOME}/conf/spark-defaults.conf
ENV PATH $PATH:${SPARK_HOME}/bin