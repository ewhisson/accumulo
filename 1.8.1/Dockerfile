FROM krallin/centos-tini:centos7
MAINTAINER Michael J. Stealey <michael.j.stealey@gmail.com>

ARG ACCUMULO_VERSION=1.8.1
ARG HADOOP_VERSION=2.9.0
ARG ZOOKEEPER_VERSION=3.4.11

# Set correct environment variables.
ENV	HOME=/root \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# java 8: https://github.com/binarybabel/docker-jdk/blob/master/src/centos.Dockerfile
ENV JAVA_DOWNLOAD=http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.rpm
RUN cd /tmp \
    && curl -o jdk.rpm -jfksSLH "Cookie: oraclelicense=accept-securebackup-cookie" \
       "${JAVA_DOWNLOAD:-$(curl -s https://lv.binarybabel.org/catalog-api/java/jdk8.txt?p=downloads.rpm)}" \
    && rpm -Uvh jdk.rpm && rm jdk.rpm \
    && echo "export JAVA_HOME=/usr/java/default/" > /etc/profile.d/java_home.sh

# apache hadoop
ARG HADOOP_INSTALL_DIR=/home/hadoop
RUN yum install -y \
    openssh-server \
    openssh-clients \
    which
RUN adduser -m -d $HADOOP_INSTALL_DIR hadoop
WORKDIR $HADOOP_INSTALL_DIR
USER hadoop
RUN curl -o hadoop-$HADOOP_VERSION.tar.gz "https://archive.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz" \
    && tar xzf hadoop-$HADOOP_VERSION.tar.gz \
    && mv hadoop-$HADOOP_VERSION hadoop \
    && rm -f hadoop-HADOOP_VERSION.tar.gz \
    && yum clean all

WORKDIR /root/
USER root
RUN ssh-keygen -q -N '' -t rsa -f /root/.ssh/id_rsa \
    && ssh-keygen -q -N '' -t dsa -f /etc/ssh/ssh_host_dsa_key \
    && ssh-keygen -q -N '' -t rsa -f /etc/ssh/ssh_host_rsa_key \
    && ssh-keygen -q -N '' -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key \
    && ssh-keygen -q -N '' -t ed25519 -f /etc/ssh/ssh_host_ed25519_key \
    && cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys \
    && chmod 0600 /root/.ssh/authorized_keys

ENV HADOOP_USER_HOME=${HADOOP_INSTALL_DIR} \
    HADOOP_PREFIX=${HADOOP_INSTALL_DIR}/hadoop \
    HADOOP_INSTALL=${HADOOP_PREFIX} \
    HADOOP_MAPRED_HOME=${HADOOP_PREFIX} \
    HADOOP_COMMON_HOME=${HADOOP_PREFIX} \
    HADOOP_HDFS_HOME=${HADOOP_PREFIX} \
    YARN_HOME=${HADOOP_PREFIX} \
    HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_PREFIX}/lib/native \
    HADOOP_CONF_DIR=${HADOOP_PREFIX}/etc/hadoop \ PATH=$PATH:${HADOOP_PREFIX}/sbin:${HADOOP_PREFIX}/bin

ENV IS_NODE_MANAGER=true \
    IS_NAME_NODE=true \
    IS_SECONDARY_NAME_NODE=true \
    IS_DATA_NODE=true \
    IS_RESOURCE_MANAGER=true \
    CLUSTER_NODES=localhost

# apache accumulo
ENV ACCUMULO_HOME=/opt/accumulo-$ACCUMULO_VERSION
RUN mkdir -p /opt/accumulo-${ACCUMULO_VERSION} \
  && curl --location --insecure --show-error "https://dist.apache.org/repos/dist/release/accumulo/${ACCUMULO_VERSION}/accumulo-${ACCUMULO_VERSION}-bin.tar.gz" > /opt/accumulo-${ACCUMULO_VERSION}.tgz \
  && tar -C /opt/accumulo-${ACCUMULO_VERSION} --extract --file /opt/accumulo-${ACCUMULO_VERSION}.tgz --strip-components=1 \
  && rm -f /opt/accumulo-${ACCUMULO_VERSION}.tgz* \
  && chown -R hadoop:hadoop ${ACCUMULO_HOME}

ENV ACCUMULO_VERSION=${ACCUMULO_VERSION} \
  ACCUMULO_PASSWORD=secret \
  ACCUMULO_INSTANCE=docker-development \
  ACCUMULO_MASTER=localhost \
  ACCUMULO_WORKERS='' \
  ZOOKEEPER_HOME=/home/hadoop/zookeeper \
  ZOOKEEPER_NODES=''

VOLUME ["/site-files", "/home/hadoop/public"]

COPY docker-entrypoint.sh /docker-entrypoint.sh
COPY zookeeper-${ZOOKEEPER_VERSION}.jar /home/hadoop/zookeeper/zookeeper-${ZOOKEEPER_VERSION}.jar

EXPOSE 22

ENTRYPOINT ["/usr/local/bin/tini", "--", "/docker-entrypoint.sh"]
