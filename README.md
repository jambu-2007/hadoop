sudo apt update
sudo apt install openjdk-8-jdk openssh-server wget -y

wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xzf hadoop-3.3.6.tar.gz
mv hadoop-3.3.6 hadoop

mkdir -p ~/hadoop/data/namenode ~/hadoop/data/datanode

echo 'export HADOOP_HOME=$HOME/hadoop' >> ~/.bashrc
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> ~/.bashrc
source ~/.bashrc

echo "export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

cat > $HADOOP_HOME/etc/hadoop/core-site.xml <<EOF
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://localhost:9000</value>
</property>
</configuration>
EOF

cat > $HADOOP_HOME/etc/hadoop/hdfs-site.xml <<EOF
<configuration>
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>$HOME/hadoop/data/namenode</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>$HOME/hadoop/data/datanode</value>
</property>
</configuration>
EOF

cat > $HADOOP_HOME/etc/hadoop/mapred-site.xml <<EOF
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
</configuration>
EOF

cat > $HADOOP_HOME/etc/hadoop/yarn-site.xml <<EOF
<configuration>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
</configuration>
EOF

mkdir -p ~/.ssh
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
sudo service ssh start

hdfs namenode -format
start-dfs.sh
start-yarn.sh
jps

echo "hello hadoop hello mapreduce" > file.txt
hdfs dfs -mkdir -p /input
hdfs dfs -put file.txt /input

hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /input /output

hdfs dfs -cat /output/part-r-00000

