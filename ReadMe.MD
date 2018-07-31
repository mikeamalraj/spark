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
    
    
