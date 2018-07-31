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
    

## 2. sbt installation

Install sbt on `analytics1` machine which is used to compile spark source code.

Download sbt,

    cd /home/serendio/downloads
    wget https://piccolo.link/sbt-0.13.17.tgz
    
Extract tar file

    tar -xzf sbt-0.13.17.tgz
    
Move extracted sbt to opt folder

    sudo mkdir /opt/sbt
    sudo mv sbt-0.13.17 /opt/sbt
    
Set environment variable in `~/.bashrc` file

    export SBT_HOME=/opt/sbt/sbt-0.13.17
    export PATH=$SBT_HOME/bin:$PATH
    
then run `source ~/.bashrc`


## 3. Maven installation

Install apache maven on `analytics1` machine which is used to compile java code (TestDataGenerator & TopicSplitter).

Download sbt,

    cd /home/serendio/downloads
    wget http://www-us.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
    
Extract tar file

    tar -xzf apache-maven-3.5.4-bin.tar.gz
    
Move extracted mvn to opt folder

    sudo mkdir /opt/maven
    sudo mv apache-maven-3.5.4 /opt/maven
    
Set environment variable in `~/.bashrc` file

    export MAVEN_HOME=/opt/maven/apache-maven-3.5.3
    export PATH=$MAVEN_HOME/bin:$PATH
    
then run `source ~/.bashrc`


## 4. Git installation

To install git run,

    sudo apt-get install git
    

## 5. Hadoop installation






    

    
