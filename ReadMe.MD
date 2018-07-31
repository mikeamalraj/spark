# Tools and Software installation


**General note**

1. Create downloads folder under home directory and put all your downloaded files in `/home/serendio/downloads`
1. Installation folder: `/opt/`
1. Add custom environment variables in `~/.bashrc` file

## 1. Java installation

Install the latest version of  Java on all three instances.

Download java,

    cd /home/serendio/downloads
    wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz
    
Extract tar file

     tar -xzf jdk-8u171-linux-x64.tar.gz
     
Move extracted java into opt foler

    sudo mkdir /opt/java
    sudo mv jdk1.8.0_171 /opt/java
    
Install java

    sudo update-alternatives --install /usr/bin/java java /opt/java/jdk1.8.0_171/bin/java 1
    sudo update-alternatives --install /usr/bin/javac javac /opt/java/jdk1.8.0_171/bin/javac 1
  
Set default java, if more than one java version installed.

    sudo update-alternatives --config java
    sudo update-alternatives --config javac

Set java environment variable. Open `~/.bashrc` file and add below lines

    export JAVA_HOME=/opt/java/jdk1.8.0_171
	export PATH=$JAVA_HOME/bin:$PATH

then run `source ~/.bashrc`.

To verify java installation, run `java -version`. 

    java version "1.8.0_171"
    Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
	Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

To verify java path, run `echo $JAVA_HOME`.

    /opt/java/jdk1.8.0_171
    
    
    

    
