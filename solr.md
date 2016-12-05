# Building Apache Solr
The instructions provided below specify the steps to build Apache Solr version 6.3.0 on the IBM z Systems for RHEL 7.1/7.2/7.3, SLES 12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_ 	

* _When following the steps below please use a standard permission user unless otherwise specified._
	 
* _A directory `/<source_root>/` will be referred to in these instructions. This is a temporary writable directory anywhere you'd like to place it._

##Step 1: Building and Installing Apache Solr

####1.1) Install dependencies  

* RHEL 7.1/7.2/7.3
  ```shell
  sudo yum install -y wget tar java-1.8.0-ibm-devel.s390x
  ```

* SLES 12-SP1/12-SP2
  ```shell
  sudo zypper install -y wget tar java-1_8_0-ibm-devel
  ```

* Ubuntu 16.04/16.10
  ```shell
  sudo apt-get install -y wget tar openjdk-8-jdk ant
  ```
  
####1.2) Download and Install ant 1.9.6 
* RHEL 7.1/7.2/7.3 and SLES 12-SP1/12-SP2

  ```
  wget http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.6-bin.tar.gz
  tar zxvf apache-ant-1.9.6-bin.tar.gz
  export PATH=/<source_root>/apache-ant-1.9.6/bin:$PATH
  ```
_**Note:** ant version available in yum and zypper is very low version. It needs to be atleast at version 1.9.6._  

####1.3) Download Apache Solr 6.3.0 source code

  ```shell
  cd /<source_root>/
  wget http://archive.apache.org/dist/lucene/solr/6.3.0/solr-6.3.0-src.tgz
  tar zxvf solr-6.3.0-src.tgz
  ```
####1.4) Set JAVA_HOME

* RHEL 7.1/7.2/7.3 and SLES 12-SP1/12-SP2 

  ```shell
  export JAVA_HOME=/etc/alternatives/java_sdk_ibm
  ```
* Ubuntu 16.04/16.10

  ```shell
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-s390x
  ```
  
####1.5) Configure and make
  ```shell
  cd /<source_root>/solr-6.3.0/solr
  ant ivy-bootstrap
  ant server
  ```

## Step 2: Verification 

####2.1) Start Solr and create a core
  ```shell
  cd /<source_root>/solr-6.3.0/solr/bin
  chmod a+x solr
  ./solr start
  ./solr create -c samp
  ```

####2.2)Expected Output 
* The solr start command will start the solr server on port 8983. `./solr create -c` samp command would create the samp as
  
  ```
  {"responseHeader":{
  "status":0,
  "QTime":720},"core":"samp"}
  ```

  _**Note:** If the solr create command fails due to any reason, please stop and start the solr server and then rerun the command._ 

* After starting Solr, direct your Web browser to the Solr Admin Console at: `http://HOST_IP:8983/solr/` 

###References:

http://lucene.apache.org/solr - Details of Apache Solr can be found here.
