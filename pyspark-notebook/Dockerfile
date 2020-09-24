# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=jupyter/scipy-notebook
FROM $BASE_CONTAINER

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

# Fix DL4006
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# Spark dependencies
# Default values can be overridden at build time
# (ARGS are in lower case to distinguish them from ENV)
ARG spark_version="3.0.1"
ARG hadoop_version="3.2"
ARG spark_checksum="E8B47C5B658E0FBC1E57EEA06262649D8418AE2B2765E44DA53AAF50094877D17297CC5F0B9B35DF2CEEF830F19AA31D7E56EAD950BBE7F8830D6874F88CFC3C"
ARG py4j_version="0.10.9"
ARG openjdk_version="11"

ENV APACHE_SPARK_VERSION="${spark_version}" \
    HADOOP_VERSION="${hadoop_version}"

# Spark installation
WORKDIR /tmp

### Only for java 9 (https://github.com/jupyter/notebook/issues/743#issuecomment-376433913)
#ENV JAVA_HOME="/usr/lib/jvm/jdk-${openjdk_version}"
#RUN wget https://download.java.net/java/GA/jdk9/${openjdk_version}/binaries/openjdk-${openjdk_version}_linux-x64_bin.tar.gz
#RUN tar -xvf openjdk-${openjdk_version}_linux-x64_bin.tar.gz && \
#    mkdir -p /usr/lib/jvm && \
#    mv jdk-9 /usr/lib/jvm/ && \
#    update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk-${openjdk_version}/bin/java" 1 && \
#    update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk-${openjdk_version}/bin/javac" 1 && \
#    chmod a+x /usr/bin/java && \
#    chmod a+x /usr/bin/javac && \
#    chown -R root:root /usr/lib/jvm/jdk-9 && \
#    rm openjdk-9_linux-x64_bin.tar.gz

RUN apt-get -y update && \
    apt-get install --no-install-recommends -y \
    "openjdk-${openjdk_version}-jre-headless" \
    ca-certificates-java && \
    rm -rf /var/lib/apt/lists/*

# Using the preferred mirror to download Spark
# hadolint ignore=SC2046
# Get spark from https://archive.apache.org/dist/spark
RUN wget -q https://archive.apache.org/dist/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
    echo "${spark_checksum} *spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" | sha512sum -c - && \
    tar xzf "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" -C /usr/local --owner root --group root --no-same-owner && \
    rm "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz"

WORKDIR /usr/local
RUN ln -s "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}" spark

# Configure Spark
ENV SPARK_HOME=/usr/local/spark
ENV PYTHONPATH="${SPARK_HOME}/python:${SPARK_HOME}/python/lib/py4j-${py4j_version}-src.zip" \
    SPARK_OPTS="--driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info" \
    PATH=$PATH:$SPARK_HOME/bin

# Install AWS packages
WORKDIR /tmp
RUN wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.7.4/aws-java-sdk-1.7.4.jar && \
    wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.7.3/hadoop-aws-2.7.3.jar && \
    mv aws-java-sdk-1.7.4.jar hadoop-aws-2.7.3.jar ${SPARK_HOME}/jars/
WORKDIR /usr/local

USER $NB_UID

# Install pyarrow
RUN conda install --quiet -y 'pyarrow' && \
    conda clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

WORKDIR $HOME
