<!---PACKAGE:OpenShift Origin--->
<!---DISTRO:RHEL 7.1:1.1.3--->

# Building OpenShift Origin

[OpenShift Origin V3](https://github.com/openshift/origin) is a distribution of Kubernetes optimized for continuous application development and multi-tenant deployment. The instructions provided below specify the steps to build OpenShift Origin 1.3.1 on Linux on the IBM z Systems for RHEL 7.2.

### _**General Notes:**_  
1. _When following the steps below please use a standard permission user unless otherwise specified._

2. _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

## Building and Installing OpenShift Origin

### Step 1: Install the Dependencies

    sudo yum install git tar which

### Step 2: Install Docker 1.10.0
See instructions [here](http://www.ibm.com/developerworks/linux/linux390/docker.html)

### Step 3: Install Go 1.7

See instructions [here](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7).  

### Step 4: Build OpenShift Origin by following steps 1 through 5
See instructions [here](https://github.com/openshift/origin/blob/master/CONTRIBUTING.adoc#develop-locally-on-your-host) to develop locally on your host

####In summary:
```
     export GOPATH=/<source_root>/go
     export PATH=$PATH:$GOPATH/bin
     export OS_OUTPUT_GOPATH=1
     mkdir -p $GOPATH/src/github.com/openshift
     cd $GOPATH/src/github.com/openshift
     git clone https://github.com/openshift/origin.git  
     cd origin  
     git checkout tags/v1.3.1 -b v1.3.1  

     cd $GOPATH/src/github.com/openshift/origin/vendor/golang.org/x/ # Replace sys package
     mv sys sys.bk
     git clone https://github.com/linux-on-ibm-z/sys.git

     cd $GOPATH/src/github.com/openshift/origin
     make clean build
```

Above steps will generate OpenShift Origin executable in the directory specified in the instructions.  You are now ready to use OpenShift Origin.

__Note:__ In order to use OpenShift Origin you must generate core OpenShift Docker images. Here we provide guidance on how to do that on RHEL 7.2. Feel free to adapt it to your environment.

### Optional: Build required Docker images

1. Build base RHEL image - see [details](http://containerz.blogspot.ca/2015/03/creating-base-images.html#more)

2. Modify dockerfiles and scripts, assuming your current working directory is `$GOPATH/src/github.com/openshift/origin/`
  - The scripts need to be modified:   

    * `Makefile`, add s390x support:  

        ```diff
        @@ -181,7 +181,7 @@ clean:
         # Example:
         #   make release
         release: clean
        -	OS_ONLY_BUILD_PLATFORMS="linux/amd64" hack/build-release.sh
        +	OS_ONLY_BUILD_PLATFORMS="linux/s390x" hack/build-release.sh
         	hack/build-images.sh
         	hack/extract-release.sh
         .PHONY: release
        ```

    * `hack/build-images.sh`, add s390x support:  

        ```diff
        @@ -12,7 +12,7 @@ source "${OS_ROOT}/contrib/node/install-sdn.sh"
 
         if [[ "${OS_RELEASE:-}" == "n" ]]; then
           # Use local binaries
        -  imagedir="${OS_OUTPUT_BINPATH}/linux/amd64"
        +  imagedir="${OS_OUTPUT_BINPATH}/linux/s390x"
           # identical to build-cross.sh
           os::build::os_version_vars
           OS_RELEASE_COMMIT="${OS_GIT_SHORT_VERSION}"
        @@ -31,7 +31,7 @@ else
           fi
         
           # Extract the release achives to a staging area.
        -  os::build::detect_local_release_tars "linux-64bit"
        +  os::build::detect_local_release_tars "linux-s390x"
 
           echo "Building images from release tars for commit ${OS_RELEASE_COMMIT}:"
           echo " primary: $(basename ${OS_PRIMARY_RELEASE_TAR})"
        ```

    * `hack/common.sh`, change some variables and add s390x support:
        ```diff
        @@ -15,7 +15,7 @@ readonly OS_OUTPUT_PKGDIR="${OS_OUTPUT}/pkgdir"
         readonly OS_GO_PACKAGE=github.com/openshift/origin
 
         readonly OS_IMAGE_COMPILE_PLATFORMS=(
        -  linux/amd64
        +  linux/s390x
         )
         readonly OS_IMAGE_COMPILE_TARGETS=(
           images/pod
        @@ -34,6 +34,7 @@ readonly OS_CROSS_COMPILE_PLATFORMS=(
           darwin/amd64
           windows/amd64
           linux/386
        +  linux/s390x
         )
         readonly OS_CROSS_COMPILE_TARGETS=(
           cmd/openshift
        @@ -442,15 +449,19 @@ function os::build::place_bins() {
                 elif [[ $platform == "linux/386" ]]; then
                   platform="linux/32bit" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
                 elif [[ $platform == "linux/amd64" ]]; then
        -          platform="linux/64bit" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
        -          platform="linux/64bit" OS_RELEASE_ARCHIVE="openshift-origin-server" os::build::archive_tar "${OS_BINARY_RELEASE_SERVER_LINUX[@]}"
        +	  platform="linux/amd64" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
        +	elif [[ $platform == "linux/s390x" ]]; then 
        +	 platform="linux/s390x" OS_RELEASE_ARCHIVE="openshift-origin-client-tools" os::build::archive_tar         "${OS_BINARY_RELEASE_CLIENT_LINUX[@]}"
        +	 platform="linux/s390x" OS_RELEASE_ARCHIVE="openshift-origin-server" os::build::archive_tar         "${OS_BINARY_RELEASE_SERVER_LINUX[@]}"
                 else
                   echo "++ ERROR: No release type defined for $platform"
                 fi
               else
        -        if [[ $platform == "linux/amd64" ]]; then
        -          platform="linux/64bit" os::build::archive_tar "./*"
        +        if [[ $platform == "linux/s390x" ]]; then
        +          platform="linux/s390x" os::build::archive_tar "./*"
                 else
                   echo "++ ERROR: No release type defined for $platform"
                 fi
               fi
        ```

    * `images/router/haproxy/reload-haproxy`, switch to `/usr/local/sbin/haproxy` for the customized haproxy installation.
        ```diff
        @@ -81,7 +81,7 @@ if [ -n "$old_pids" ]; then
             fi
           fi
 
        -  /usr/sbin/haproxy -f $config_file -p $pid_file -sf $old_pids
        +  /usr/local/sbin/haproxy -f $config_file -p $pid_file -sf $old_pids
           reload_status=$?
 
           if [[ "$installed_iptables" == 1 ]]; then
        @@ -108,7 +108,7 @@ if [ -n "$old_pids" ]; then
             fi
           fi
         else
        -  /usr/sbin/haproxy -f $config_file -p $pid_file
        +  /usr/local/sbin/haproxy -f $config_file -p $pid_file -sf $old_pids 
           reload_status=$?
         fi
        ```

    * `images/router/haproxy/conf/haproxy-config.template`:

        ```diff
        @@ -69,9 +69,11 @@ defaults
         {{ end }}
 
         {{ if (gt .StatsPort 0) }}
        -listen stats :{{.StatsPort}}
        +listen stats
        +   bind :{{.StatsPort}}
         {{ else }}
        -listen stats :1936
        +listen stats
        +    bind :1936
         {{ end }}
             mode http
             # Health check monitoring uri.
        ```

  - The docker files need to be changed: 

    * `images/base/Dockerfile`, all images should be based on our rhel7 base, so package installation should follow the rhel flavor, e.g., `yum install`:  

        ```diff
        @@ -4,14 +4,20 @@
         #
         # The standard name for this image is openshift/origin-base
         #
        -FROM centos:centos7
        +FROM rhel7.2
 
        -RUN INSTALL_PKGS="which git tar wget hostname sysvinit-tools util-linux bsdtar epel-release \
        -      socat ethtool device-mapper iptables tree findutils nmap-ncat e2fsprogs xfsprogs lsof" && \

        +RUN INSTALL_PKGS="which git tar wget hostname sysvinit-tools util-linux bsdtar \
        +     socat ethtool tree findutils nmap-ncat e2fsprogs xfsprogs lsof" && \
              yum install -y $INSTALL_PKGS && \
              rpm -V $INSTALL_PKGS && \
              yum clean all && \
              mkdir -p /var/lib/origin
        ```

    * `images/builder/docker/custom-docker-builder/Dockerfile`, add scripts for docker installation on s390x:

        ```diff
        @@ -16,10 +16,16 @@
        #
        FROM openshift/origin-base

        -RUN INSTALL_PKGS="gettext automake make docker" && \
        +RUN INSTALL_PKGS="gettext automake make" && \
              yum install -y $INSTALL_PKGS && \
              rpm -V $INSTALL_PKGS && \
              yum clean all
        
        +# Install docker
        +RUN mkdir /docker && cd /docker && \
        +    wget ftp://ftp.unicamp.br/pub/linuxpatch/s390x/redhat/rhel7.2/docker-1.10.1-rhel7.2-20160408.tar.gz && \
        +    tar -xvzf docker-1.10.1-rhel7.2-20160408.tar.gz && cp docker-1.10.1-rhel7.2-20160408/docker /usr/bin

        LABEL io.k8s.display-name="OpenShift Origin Custom Builder Example" \
       io.k8s.description="This is an example of a custom builder for use with OpenShift Origin."

        ```
    * `images/ipfailover/keepalived/Dockerfile`, add build steps to install keepalive on s390x.
        ```diff  
        @@ -5,10 +5,17 @@
         #
         FROM openshift/origin
 
        -RUN INSTALL_PKGS="kmod keepalived iproute psmisc nmap-ncat net-tools" && \
        +RUN INSTALL_PKGS="kmod iproute psmisc nmap-ncat net-tools wget tar git gcc openssl-devel make" && \
             yum install -y $INSTALL_PKGS && \
             rpm -V $INSTALL_PKGS && \
             yum clean all
        +
        +# Build and install keepalived
        +RUN git clone https://github.com/acassen/keepalived.git && \
        +    cd keepalived && \
        +    git checkout v1.2.13 && \
        +    ./configure && make && make install
        +
         COPY . /var/lib/ipfailover/keepalived/
         
         LABEL io.k8s.display-name="OpenShift Origin IP Failover" \
        ```

    * `images/node/Dockerfile`, add scripts for openvswitch installation on s390x.
        ```diff
        @@ -16,16 +16,23 @@ COPY lib/systemd/system/docker.service.d/docker-sdn-ovs.conf /usr/lib/systemd/sy
        COPY scripts/* /usr/local/bin/
 
        RUN curl -L -o /etc/yum.repos.d/origin-next-epel-7.repo https://copr.fedoraproject.org/coprs/maxamillion/origin-next/repo/epel-7/maxamillion-origin-next-epel-7.repo && \
        -    INSTALL_PKGS="libmnl libnetfilter_conntrack openvswitch \
        -      libnfnetlink iptables iproute bridge-utils procps-ng ethtool socat openssl \
        +    INSTALL_PKGS="wget tar make gcc openssl python perl libmnl libnetfilter_conntrack \
        +      libnfnetlink iptables iproute bridge-utils procps-ng ethtool socat openssl \
               binutils xz kmod-libs kmod sysvinit-tools device-mapper-libs dbus \
        -      ceph-common iscsi-initiator-utils" && \
        +      iscsi-initiator-utils" && \
              yum install -y $INSTALL_PKGS && \
              rpm -V $INSTALL_PKGS && \
              yum clean all && \
              mkdir -p /usr/lib/systemd/system/origin-node.service.d /usr/lib/systemd/system/docker.service.d && \
              chmod +x /usr/local/bin/* /usr/bin/openshift-*
 
        +RUN mkdir openvswitch && cd ./openvswitch
        +RUN wget http://openvswitch.org/releases/openvswitch-2.4.0.tar.gz
        +RUN tar -xvf openvswitch-2.4.0.tar.gz
        +RUN cd openvswitch-2.4.0/ && ./configure && make && make install
        +
        LABEL io.k8s.display-name="OpenShift Origin Node" \
        io.k8s.description="This is a component of OpenShift Origin and contains the software for individual nodes when using SDN."
       VOLUME /etc/origin/node
        ```

    * `images/openvswitch/Dockerfile`, add scripts for openvswitch installation on s390x.
        ```diff
        @@ -7,15 +7,30 @@ FROM openshift/origin-base
 
        COPY scripts/* /usr/local/bin/
 
        -RUN curl -L -o /etc/yum.repos.d/origin-next-epel-7.repo https://copr.fedoraproject.org/coprs/maxamillion/origin-next/repo/epel-7/maxamillion-origin-next-epel-7.repo && \
        -    INSTALL_PKGS="openvswitch" && \
        -    yum install -y $INSTALL_PKGS && \
        -    rpm -V $INSTALL_PKGS && \
        -    yum clean all && \
        +
        +RUN yum install -y wget tar make gcc openssl python perl && yum clean all 
        +
        +# Install openvswitch from the source
        +RUN mkdir openvswitch && cd ./openvswitch && \
        +    wget http://openvswitch.org/releases/openvswitch-2.4.0.tar.gz && \
        +    tar -xvf openvswitch-2.4.0.tar.gz && 
        +    cd openvswitch-2.4.0/ && ./configure && make && make install
        +
        +RUN    yum clean all && \
             chmod +x /usr/local/bin/*
 
        LABEL io.k8s.display-name="OpenShift Origin OpenVSwitch Daemon" \
       io.k8s.description="This is a component of OpenShift Origin and runs an OpenVSwitch daemon process."
        VOLUME /etc/openswitch
        ENV HOME /root
        +
        +ADD  scripts/* /usr/local/bin/
        +RUN chmod +x /usr/local/bin/*
        +
        ENTRYPOINT ["/usr/local/bin/ovs-run.sh"]
        ```

    * `images/release/Dockerfile`, add scripts for go installation on s390x. 

        __Note:__ We copy the compiled go directory to the image directly. 

        ```diff
        @@ -19,8 +19,13 @@ RUN mkdir $TMPDIR && \
             yum install -y $INSTALL_PKGS && \
             rpm -V $INSTALL_PKGS && \
             yum clean all && \
        -    curl https://storage.googleapis.com/golang/go$VERSION.linux-amd64.tar.gz | tar -C /usr/local -xzf - && \
        -    go get golang.org/x/tools/cmd/cover github.com/tools/godep github.com/golang/lint/golint && \
        +    wget https://storage.googleapis.com/golang/go1.7.1.linux-s390x.tar.gz && \
        +    chmod ugo+r go1.7.1.linux-s390x.tar.gz && \
        +    tar -C /usr/local -xzf go1.7.1.linux-s390x.tar.gz && \
        +    cd /usr/bin && \
        +    ln gcc s390x-linux-gnu-gcc
        +
        +    RUN go get golang.org/x/tools/cmd/cover github.com/tools/godep github.com/golang/lint/golint && \
             touch /os-build-image
 
         # Allows building Origin sources mounted using volume
        ```

    * `images/router/haproxy-base/Dockerfile`, add scripts for haproxy installation on s390x.
        ```diff
        @@ -11,10 +11,29 @@ FROM openshift/origin-base
         #       this is temporary and should be removed when the container is switch to an empty-dir
         #       with gid support.
         #
        -RUN INSTALL_PKGS="haproxy iptables lsof" && \
        +RUN INSTALL_PKGS="git  java-1.8.0-openjdk gcc-c++ tar  openssl  \
        +       openssl-devel pcre pcre-devel make iptables lsof" && \
             yum install -y $INSTALL_PKGS && \
        -    rpm -V $INSTALL_PKGS && \
        -    mkdir -p /var/lib/containers/router/{certs,cacerts} && \
        +    rpm -V $INSTALL_PKGS
        +
        +# Build and install Node.js
        +RUN wget http://www.haproxy.org/download/1.6/src/haproxy-1.6.9.tar.gz && tar xzvf haproxy-1.6.9.tar.gz && \
        +    cd haproxy-1.6.9 && make TARGET=linux26 USE_OPENSSL=1 && make install

        +RUN mkdir -p /var/lib/containers/router/{certs,cacerts} && \
             mkdir -p /var/lib/haproxy/{conf,run,bin,log} && \
             touch /var/lib/haproxy                        /conf/{{os_http_be,os_edge_http_be,os_tcp_be,os_sni_passthrough,os_reencrypt,os_edge_http_expose,os_edge_http_redirect}.map,haproxy.config} && \
             chmod -R 777 /var && \
        ```

    * `images/router/haproxy/Dockerfile`.
        ```diff
        @@ -10,15 +10,31 @@ FROM openshift/origin
         #       this is temporary and should be removed when the container is switch to an empty-dir
         #       with gid support.
         #
        -RUN INSTALL_PKGS="haproxy" && \
        +RUN INSTALL_PKGS="git java-1.8.0-openjdk gcc-c++ tar openssl openssl-devel pcre \
        +   pcre-devel make" && \
            yum install -y $INSTALL_PKGS && \
            rpm -V $INSTALL_PKGS && \
        -    yum clean all && \
        +    yum clean all
        -    mkdir -p /var/lib/haproxy/router/{certs,cacerts} && \
        -    mkdir -p /var/lib/haproxy/{conf,run,bin,log} && \

        +# Build and install Node.js
        +RUN wget http://www.haproxy.org/download/1.6/src/haproxy-1.6.9.tar.gz && tar xzvf haproxy-1.6.9.tar.gz && \
        +cd haproxy-1.6.9 && make TARGET=linux26 USE_OPENSSL=1 && make install
        
        +RUN  mkdir -p /var/lib/haproxy/router/{certs,cacerts} && \
        +     mkdir -p /var/lib/haproxy/{conf,run,bin,log} && \
              touch /var/lib/haproxy        /conf/{{os_http_be,os_edge_http_be,os_tcp_be,os_sni_passthrough,os_reencrypt,os_edge_http_expose,os_edge_http_redirect}.map,haproxy.config} && \
             chmod -R 777 /var && \
        -    setcap 'cap_net_bind_service=ep' /usr/sbin/haproxy
        +    setcap 'cap_net_bind_service=ep' /usr/local/sbin/haproxy
 
         COPY . /var/lib/haproxy/
        ```

3. Build base images 
 
    __Note:__ If you want to build your own release, you have to git commit all changes back to your git server so that no dirty version remains.
    ```
    cd $GOPATH/src/github.com/openshift/origin/
    hack/build-base-images.sh
    ```

4. Build release 
   
   Build below Dockerfile explicitly with tag `openshift/origin-release:golang-1.6` for s390x before running `make release` command.

    ```
    #
    # This is the image that controls the standard build environment for releasing OpenShift Origin.
    # It is responsible for performing a cross platform build of OpenShift.
    #
    # The standard name for this image is openshift/origin-release
    #
    FROM openshift/origin-base

    ENV VERSION=1.6 \
    GOARM=5 \
    GOPATH=/go \
    GOROOT=/usr/local/go \
    OS_VERSION_FILE=/go/src/github.com/openshift/origin/os-version-defs \
    TMPDIR=/openshifttmp

    ENV PATH=$PATH:$GOROOT/bin:$GOPATH/bin

    RUN mkdir $TMPDIR && \
        INSTALL_PKGS="make gcc zip mercurial krb5-devel bsdtar" && \
        yum install -y $INSTALL_PKGS && \
        rpm -V $INSTALL_PKGS && \
        yum clean all && \
        wget https://storage.googleapis.com/golang/go1.7.1.linux-s390x.tar.gz && \
        chmod ugo+r go1.7.1.linux-s390x.tar.gz && \
        tar -C /usr/local -xzf go1.7.1.linux-s390x.tar.gz && \
        cd /usr/bin && \
        ln gcc s390x-linux-gnu-gcc && \
        go get golang.org/x/tools/cmd/cover github.com/tools/godep github.com/golang/lint/golint && \
        touch /os-build-image

    WORKDIR /go/src/github.com/openshift/origin
    LABEL io.k8s.display-name="OpenShift Origin Release Environment (golang-$VERSION)" \
          io.k8s.description="This is the standard release image for OpenShift Origin and contains the necessary build tools to build the platform."

     # Expect the working directory to be populated with the repo source
    CMD hack/build-cross.sh
    ```

    and now run  

    ```
    make release
    ```

5. If you intend to use Auto-scaling feature of OpenShift Origin then OpenShift [origin-metrics](https://github.com/openshift/origin-metrics) images are required to be built:
  - Git clone origin-metrics in a directory of your choosing: 
    ```
      cd /<source_root>/
      git clone https://github.com/openshift/origin-metrics.git
      cd /<source_root>/origin-metrics
      git checkout v1.3.1
    ```

  - Create [wildfly](https://hub.docker.com/r/jboss/wildfly/) images required by origin-metrics. You will also need to build dependent Docker images for wildfly. Dockerfile of hawkular-metrics uses this docker image as base image and should look like as follow:

    ```diff
    @@ -20,7 +20,7 @@
     # This dockerfile can be used to create a Hawkular-Metrics docker
     # image to be run on Openshift.

    -FROM jboss/wildfly:10.0.0.Final
    +FROM wildfly

     #TODO: remove this section once wildfly:10.1.0.CR1 is available in a docker image
     ENV WILDFLY_SHA1 e44dffaff2ed88e834dfcb592b8473083089a56d
    @@ -80,9 +80,14 @@ COPY logging.properties $JBOSS_HOME/standalone/configuration/logging.properties

     # Install Hawkular-Metrics Python client, only available on EPEL
     USER root
    -RUN yum -y install epel-release \
    -    && yum -y install python-pip \
    -    && yum clean all \
    +
    +RUN yum install -y wget && \
    +    wget https://bootstrap.pypa.io/get-pip.py && \
    +    python get-pip.py && \
    +    yum clean all \
         && pip install --no-cache-dir 'hawkular-client==0.4.4'

    # Change the permissions so that the user running the image can start up Hawkular Metrics
    ```

  - Modify Cassandra Dockerfile to include Linux on Z system changes. You will need to integrate Cassandra changes as per [Apache Cassandra 3.x](https://github.com/linux-on-ibm-z/docs/wiki/Building-Apache-Cassandra)

  - Modify Heapster Base dockerfile to remove `go` dependency from 'yum install ..' command and add `gcc` dependency instead. Also modify it to replace Go installation steps by the steps mentioned in [Go 1.7](https://github.com/linux-on-ibm-z/docs/wiki/Building-Go-1.7) recipe.

  - You will need to build dependent Docker images for Heapster Base. Heapster dockerfile uses this docker image as base image.

  - Modify `deployer` dockerfile to add created `openshift/origin:latest` as base image:
    ```diff
    @@ -15,7 +15,7 @@
     # limitations under the License.
     #

    -FROM openshift/origin:v1.3.0-alpha.3
    +FROM openshift/origin:latest

     # The image is maintained by the Hawkular Metrics team
    ```

  - Run `hack/build-images.sh` to create origin-metrics images

  
### Optional: Run all tests including unit, integration, etc.  

  * Modify `hack/test-go.sh` to remove `-race` check for s390x:  

    ```diff
    @@ -70,8 +70,8 @@ fi
     # Internalize environment variables we consume and default if they're not set
     dry_run="${DRY_RUN:-}"
     test_kube="${TEST_KUBE:-}"
     test_timeout="${TIMEOUT:-120s}"
    -detect_races="${DETECT_RACES:-true}"
    +detect_races="${DETECT_RACES:-}"
     coverage_output_dir="${COVERAGE_OUTPUT_DIR:-}"
     coverage_spec="${COVERAGE_SPEC:--cover -covermode atomic}"
     gotest_flags="${GOTEST_FLAGS:-}"
    @@ -288,5 +288,11 @@ elif [[ -n "${dlv_debug}" ]]; then
         dlv test ${test_packages}
    else
       # we need to generate neither jUnit XML nor coverage reports
        go test ${gotest_flags} ${test_packages}
    fi
    ```

  * Add support for s390x in `kubelet_test.go` to fix _nodes_ related test case failures. For this, modify `./vendor/k8s.io/kubernetes/pkg/kubelet/kubelet_test.go` as follows:  


    ```diff
    @@ -32,6 +32,7 @@ import (
            "testing"
            "time"

    +       goruntime "runtime"
            cadvisorapi "github.com/google/cadvisor/info/v1"
            cadvisorapiv2 "github.com/google/cadvisor/info/v2"
            "github.com/stretchr/testify/assert"
    @@ -2801,8 +2802,8 @@ func TestUpdateNewNodeStatus(t *testing.T) {
                                    BootID:                  "1b3",
                                    KernelVersion:           "3.16.0-0.bpo.4-amd64",
                                    OSImage:                 "Debian GNU/Linux 7 (wheezy)",
    -                               OperatingSystem:         "linux",
    -                               Architecture:            "amd64",
    +                               OperatingSystem:         goruntime.GOOS,
    +                               Architecture:            goruntime.GOARCH,
                                    ContainerRuntimeVersion: "test://1.5.0",
                                    KubeletVersion:          version.Get().String(),
                                    KubeProxyVersion:        version.Get().String(),
    @@ -3049,7 +3050,7 @@ func TestUpdateExistingNodeStatus(t *testing.T) {
                                    KernelVersion:           "3.16.0-0.bpo.4-amd64",
                                    OSImage:                 "Debian GNU/Linux 7 (wheezy)",
    -                               OperatingSystem:         "linux",
    -                               Architecture:            "amd64",
    +                               OperatingSystem:         goruntime.GOOS,
    +                               Architecture:            goruntime.GOARCH,
                                    ContainerRuntimeVersion: "test://1.5.0",
                                    KubeletVersion:          version.Get().String(),
                                    KubeProxyVersion:        version.Get().String(),
    @@ -3332,8 +3333,8 @@ func TestUpdateNodeStatusWithRuntimeStateError(t *testing.T) {
                                    BootID:                  "1b3",
                                    KernelVersion:           "3.16.0-0.bpo.4-amd64",
                                    OSImage:                 "Debian GNU/Linux 7 (wheezy)",
    -                               OperatingSystem:         "linux",
    -                               Architecture:            "amd64",
    +                               OperatingSystem:         goruntime.GOOS,
    +                               Architecture:            goruntime.GOARCH,
                                    ContainerRuntimeVersion: "test://1.5.0",
                                    KubeletVersion:          version.Get().String(),
                                    KubeProxyVersion:        version.Get().String(),

    ```

  * Run all tests  

      ```
      make test PERMISSIVE_GO=y
      ```

__Notes:__ 
  1. PERMISSIVE_GO=y argument is used to bypass a check for Go 1.6.  
  2. There is one failing test case regarding HTTPProbeChecker, etc. which we can skip for this version. This is believed to be fixed in the newer version of OpenShift.
  3. If in case you require to rerun `make test` command, run `make update` first to update to clean up cache.
  4. If in case while running `oc cluster up` command, it starts pulling `openshift/origin:v1.3.1`, tag already created `openshift/origin:latest` image with `openshift/origin:v1.3.1`.
   
## Reference
https://github.com/openshift/origin
