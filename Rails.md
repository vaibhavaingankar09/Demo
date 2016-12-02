<!---PACKAGE:Rails--->
<!---DISTRO:RHEL 6.6:5.0.0--->
<!---DISTRO:RHEL 7.1:5.0.0--->
<!---DISTRO:SLES 11:5.0.0--->
<!---DISTRO:SLES 12:5.0.0--->
<!---DISTRO:Ubuntu 16.x:5.0.0--->

# Building Ruby on Rails

Below versions of Ruby on Rails are available in respective distributions at the time of this recipe creation:

  * Ubuntu 16.04 has `4.2.6`
  * Ubuntu 16.10 has `4.2.7`

The instructions provided below specify the steps to build Rails version 5.0.0.1 on IBM z Systems for RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4, SLES 12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.  

_**General Notes:**_   
* _When following the steps below please use a standard permission user unless otherwise specified._     
* _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

##Step 1: Build Ruby
####1.1) Build Ruby 1.9.3+ 
 * [Building Ruby](https://github.com/linux-on-ibm-z/docs/wiki/Building-Ruby) (For RHEL6.8, RHEL 7.1/7.2/7.3, SLES11-SP4 and SLES 12/12-SP1/12-SP2)

_**Note:** The available yum / zypper version of Ruby is too low level for the distros._
####1.2) Correct the gem environment for a standard user

  * RHEL 6.8, RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2
		
          export GEM_HOME=/home/<USER>/.gem/ruby
          export PATH=/home/<USER>/.gem/ruby/bin:$PATH
		
_**Note:** Where `<USER>` is the standard user you are installing under._

##Step 2: Installing Ruby on Rails
####2.1) Add build dependencies
    
   * RHEL 6.8 and RHEL 7.1/7.2/7.3

          sudo yum install -y patch make gcc
    
   * SLES 11-SP4 and SLES 12/12-SP1/SP2

          sudo zypper install -y patch make gcc
        
   * Ubuntu 16.04/16.10
 
          sudo apt-get install -y ruby ruby-dev patch make gcc zlib1g-dev
 

####2.2) Install Ruby on Rails via gem

   * RHEL 6.8 and RHEL 7.1/7.2/7.3, SLES 11-SP4 and SLES 12/12-SP1/12-SP2
      
          gem install rails -v 5.0.0.1
          
   * Ubuntu 16.04/16.10
           
          sudo gem install rails -v 5.0.0.1
    
####2.3) Ruby on Rails is now installed. Verify version with command `rails -v`
    
   * (Output)
      
          5.0.0.1
     
##References:	 
* http://rubyonrails.org/
