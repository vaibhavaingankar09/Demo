# Building htop

htop is available on Ubuntu 16.10. Should you require a different version from those listed below you will need to build it.

*    Ubuntu 16.04 has `2.0.2`

The instructions provided below specify the steps to build htop version 2.0.2 on Linux on the IBM z Systems for RHEL 6.8/7.1/7.2, SLES 11-SP4/12/12-SP1 and Ubuntu 16.04.

_**General Notes:**_

* _When following the steps below please use a standard permission user unless otherwise specified._

* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### 1.   Install the dependencies 

* RHEL 6.8/7.1/7.2
    
  ```sh
   sudo yum install ncurses ncurses-devel gcc make wget tar 
  ```

* SLES 11-SP4/12/12-SP1
    
  ```sh
   sudo zypper install ncurses ncurses-devel gcc make wget tar 
  ```

* Ubuntu 16.04
    
  ```sh
   sudo apt-get install gcc make wget tar libncursesw5 libcunit1-ncurses libncursesw5-dev
  ```

### 2. Download and unpack the htop 2.0.2 source code
```
    cd /<source_root>/
    wget https://hisham.hm/htop/releases/2.0.2/htop-2.0.2.tar.gz
    tar xvzf htop-2.0.2.tar.gz
```
### 3. Configure and build htop-2.0.2
```
	cd /<source_root>/htop-2.0.2
    ./configure
    make
```
### 4. Run test cases if any(optional)
```
    make check
```
 
### 5. Install htop
```
    sudo  make install
```

### 6. Launch htop to monitor system
```
    htop 
```
  _**Note:** For a list of supported key commands see the manual page `man htop` on ubuntu._
  
### References:
* https://hisham.hm/htop/
