使用PPA在Ubuntu上安装 Oracle Java 7
=====
以前安装JDK都是自己去Oracle下载所需的JDK版本，然后解压，然后设置环境变量。如果你有在不同版本JDK切换的需求，当每次切换时还要动手修改~/.bashrc文件。

例如：
#Setting Java environment variables
export JAVA_HOME=/home/zjh/Downloads/jdk1.7.0_45
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
export PATH=$JAVA_HOME/bin:$PATH 

##参考
http://www.webupd8.org/2012/01/install-oracle-java-jdk-7-in-ubuntu-via.html
