<!---PACKAGE:Protobuf--->
<!---DISTRO:SLES 12:3.0.0--->
<!---DISTRO:SLES 11:3.0.0--->
<!---DISTRO:RHEL 7.1:3.0.0--->
<!---DISTRO:RHEL 6.6:3.0.0--->
<!---DISTRO:Ubuntu 16.x:3.0.0--->

# Building Protobuf

Below versions of Protobuf are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `2.6.1`
*    Ubuntu 16.10 has `3.0.0`

The instructions provided below specify the steps to build Google Protobuf v3.1.0 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10. (Protobuf is available at https://developers.google.com/protocol-buffers/ and the github repository can be found at https://github.com/google/protobuf/):

_**General notes:**_ 

* When following the steps below please use a standard permission user unless otherwise specified.  
* A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it

##Step 1: Building Protobuf 3.1.0

####1.1) Install the build time dependencies


  * RHEL 6.8
    ```shell
    sudo yum install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel bison flex  binutils-devel
    ```

  * RHEL 7.1/7.2/7.3
  ```shell
  sudo yum install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel 
  ```

  * SLES 11-SP4 and SLES 12/12-SP1/12-SP2 
  ```shell
  sudo zypper install tar wget autoconf libtool automake gcc-c++ make git bzip2 curl unzip zlib zlib-devel
  ```

  * Ubuntu 16.04/16.10
  ```shell
  sudo apt-get install tar wget autoconf libtool automake g++ make git bzip2 curl unzip zlib1g-dev
  ```

You may already have these packages installed - just install any missing.

####1.2)For RHEL 6.8 and SLES 11-SP4 you will need to build Gcc 4.8+ directly

  _**Note:** The gcc version available on the RHEL 6.8 and SLES 11-SP4 package repositories is too low level. It needs to be at least at version 4.8, so build and install gcc yourself by following these steps._
	  
* Download and extract gcc
		
  ```shell
  cd /<source_root>/
  wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.2/gcc-4.8.2.tar.bz2
  bunzip2 gcc-4.8.2.tar.bz2
  tar xvf gcc-4.8.2.tar
  cd gcc-4.8.2/
  ```
* Automatically download all missing prerequisites

  ```shell
  ./contrib/download_prerequisites
  ```
* Create a build directory and configure gcc to build

  ```shell
  cd /<source_root>/
  mkdir gccbuild
  cd gccbuild/
  ../gcc-4.8.2/configure --prefix=$HOME/install/gcc-4.8.2 --enable-shared --disable-multilib --enable-threads=posix --with-system-zlib --enable-languages=c,c++
  ```
 _**Note:** You can change the `-prefix=$HOME/install/gcc-4.8.2` directory to be a different installation location if you prefer, however please note the path entry will need changing later_

* Build and install gcc

  ```shell
  make
  make install
  ```
* Update the path to match the new gcc version

  ```shell
  export PATH=$HOME/install/gcc-4.8.2/bin:$PATH
  ```
 _**Note:** If you changed the directory in `configure`, ensure it matches above._

####1.3) Clone the protobuf repository and checkout the correct version

  ```shell
  cd /<source_root>/
  git clone https://github.com/google/protobuf.git
  cd protobuf
  git checkout v3.1.0
  ```

####1.4) Generate and then run the configuration

  ```shell
  ./autogen.sh
  ./configure
  ```
  
####1.5) Build and test

  ```shell
  make
  make check
  ```
 _**Note:** There are 7 tests/suites.  All 7 should pass_

####1.6) Install Protobuf and verify the installation

  ```shell
  sudo make install
  export LD_LIBRARY_PATH=/usr/local/lib
  protoc --version
  ```
  _**Note:** Protobuf should report version `libproto 3.1.0`_

## References:
  https://developers.google.com/protocol-buffers/  
  https://github.com/google/protobuf/
