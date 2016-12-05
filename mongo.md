#Building MongoDB

The instructions provided below specify the steps to build MongoDB version 3.3.12 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.

_**General Notes:**_  
* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

* Building MongoDB requires a lot of disk space. Before attempting the build, ensure that you have 30GB of free space. You will also need root access to perform some of the steps below, e.g. installing libraries into system locations.

##Step 1: Building and Installing 
####1.1) Install the dependencies
* RHEL 6.8
  ```
  sudo yum install wget tar make flex gcc gcc-c++ binutils-devel bzip2 glibc-devel git subversion glibc-devel binutils-devel unzip zip texinfo openssl-devel
  ```

* RHEL 7.1/7.2/7.3
  ```
  sudo yum install -y wget tar make flex gcc gcc-c++ binutils-devel bzip2 glibc-devel.s390 git subversion glibc-devel.s390x binutils-devel.s390x unzip zip texinfo openssl-devel.s390
  ```

* SLES 11-SP4 and SLES 11/12-SP1/12-SP2
  ```       
  sudo zypper install wget tar make flex gcc gcc-c++ binutils-devel bzip2 git subversion binutils-devel.s390x unzip zip texinfo glibc-devel glibc-devel-32bit python-xml python-devel gawk
  ```
  
* Ubuntu 16.04
  ```       
  sudo apt-get install -y wget tar make flex gcc g++ zlib1g-dev git libssl-dev binutils binutils-dev subversion glibc-source build-essential bzip2 texinfo
  ```
  
* Ubuntu 16.10
  ```       
  sudo apt-get install -y wget tar make flex gcc-5 g++-5 zlib1g-dev git libssl-dev
  sudo ln -sf /usr/bin/g++-5 /usr/bin/g++
  sudo ln -sf /usr/bin/gcc-5 /usr/bin/gcc
  ```  
 
####1.2)Install GCC 5.3
* MongoDB 3.3.12 requires GCC 5.3

  ```
  cd /<source_root>/
  git clone https://github.com/gcc-mirror/gcc
  cd gcc
  git checkout gcc-5_3_0-release
  ./contrib/download_prerequisites
  ./configure --prefix="/opt/gcc"  --enable-shared --with-system-zlib --enable-threads=posix --enable-__cxa_atexit --enable-checking --enable-gnu-indirect-function --enable-languages="c,c++" --disable-bootstrap --disable-multilib
  make
  sudo make install
  sudo ln -sf /opt/gcc/bin/gcc /usr/bin/gcc
  sudo ln -sf /opt/gcc/bin/g++ /usr/bin/g++
  ```

####1.3) Install Scons
* RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2

   Building MongoDB requires SCons-2.3.5.1. Download from [SCons project page](http://superb-sea2.dl.sourceforge.net/project/scons/scons/2.3.5/scons-2.3.5-1.noarch.rpm) and install it like this:
  ```
  cd /<source_root>/
  wget http://superb-sea2.dl.sourceforge.net/project/scons/scons/2.3.5/scons-2.3.5-1.noarch.rpm
  sudo rpm -i scons-2.3.5-1.noarch.rpm
  ```
  
* Ubuntu 16.04/16.10 
  ```
  sudo apt-get install scons
  ```
  
####1.4) Clone MongoDB 3.3.12
  ```
  cd /<source_root>/
  git clone https://github.com/mongodb/mongo
  cd mongo
  git checkout r3.3.12
  ```
####1.5) Build MongoDB
  ```
  cd /<source_root>/mongo
  scons --opt --allocator=system all --use-s390x-crc32=off
  ```
##Step 2: Build MongoDB tools

####2.1)Clone Mongo-tools
Since version 3.0, tools such as mongodump, mongoexport, mongoimport and mongorestore have been rewritten in Go and moved to a different GitHub repository. 

* Clone the source code and check out release r3.3.11

  ```
  cd /<source_root>/
  git clone https://github.com/mongodb/mongo-tools
  cd mongo-tools
  git checkout r3.3.11
  ```

####2.2) Install Go
 * Refer [go](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7) recipe  
 * For Ubuntu, use `sudo apt-get install golang` to install the package from the repository.


####2.3) Run the script to build MongoDB tools:
  ```
  cd /<source_root>/mongo-tools
  ./build.sh
  ```

##Step 3:Testing (Optional)

To run self-verifying tests, two Python modules [PyMongo](http://api.mongodb.org/python/current/) and [PyYAML](http://pyyaml.org/wiki/PyYAML) must be installed.

   To check if you have those two modules installed, run:

        python -c "import pymongo; import yaml"

   If both modules are installed already, nothing will be printed to the console. Otherwise, the console will print an error message, such as "No module named yaml" or "No module named pymongo".

####3.1) Install PyMongo

* RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2

  ```
  cd /<source_root>/
  git clone git://github.com/mongodb/mongo-python-driver.git pymongo
  cd pymongo
  sudo python setup.py install
  ```
* Ubuntu 16.04/16.10 

  ```
  sudo apt-get -y install python-pip
  pip install --upgrade pip
  sudo python -m pip install pymongo
  ```  

####3.2) Install PyYAML

* RHEL 6.8, RHEL 7.1/7.2/7.3

  ```
  sudo yum install PyYAML
  ```
* SLES 11-SP4 and SLES 12/12-SP1/12-SP2

  ```
  cd /<source_root>/
  git clone https://github.com/yaml/pyyaml.git
  cd pyyaml
  sudo python setup.py install
  ```
* Ubuntu 16.04/16.10 

  ```
  sudo python -m pip install pyyaml
  ```  
  
###3.3)Additional dependencies required for testcases 
* Ubuntu 16.10
  ```
  sudo apt-get install -y binutils binutils-dev glibc-source build-essential bzip2 texinfo
  ```
_**Note:** The above dependencies are required for running tests after MongoDB is build on Ubuntu 16.10 only.The dependencies change the GCC version to 6.2.0._  
####3.2) There are 3 main categories of tests:

* JavaScript integration tests:
  ```
  cd /<source_root>/mongo
  python buildscripts/resmoke.py --suites=core
  sudo python buildscripts/resmoke.py --suites=core
  ```
* "New-style" C++ unit tests

  ```
  cd /<source_root>/mongo
  python buildscripts/resmoke.py --suites=unittests
  ```
* "Old-style" (deprecating) C++ dbtests

  ```
  cd /<source_root>/mongo
  ./dbtest
  ```
   More details about testing can be found at [MongoDB Manual](https://docs.mongodb.org/manual/contributors/tutorial/test-the-mongodb-server/).

##Step 4:Installation

* The binaries will be output in the `mongo/` and `mongo-tools/bin/` directories. To install them properly, execute these commands.

 ```
  for i in mongo mongobridge mongod mongoperf mongos ; do
  	cp mongo/$i /usr/local/bin/
  	chmod 755 /usr/local/bin/$i
  done

  for i in mongodump mongoexport mongofiles mongoimport \
          mongooplog mongorestore mongostat mongotop ; do
      cp mongo-tools/bin/$i /usr/local/bin/
      chmod 755 /usr/local/bin/$i
  done
  ```
## References

- [MongoDB manual](https://github.com/mongodb/mongo/wiki/Build-Mongodb-From-Source)
- [More information on running MongoDB for the first time](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-linux/#run-mongodb)
