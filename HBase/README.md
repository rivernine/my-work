# HBase + Phoenix on Windows 10

## 0. 목표
  ### **Apache HBase table data(.csv) export. (Using Apache Phoenix)**

## 1. Info
  **Apache Hbase**
  - Hadoop Platform을 위한 비관계형 분산 데이터베이스
  - Java로 제작
  - HDFS위에서 동작
  - HBase Table은 Mapreduce를 위한 입출력을 제공, Java API, REST로 접근 가능

  **Apache Phoenix**
  - NoSQL인 HBase 데이터에 대해 SQL 문법으로 쿼리 가능
  - HBase 데이터에 대한 빠른 접근이 가능
  - 추가적인 서버를 사용하지 않아 가벼움

## 2. HBase Prerequisites
  * Java
  * Hadoop
  * Zookeeper

     **[> 자세한 사항은 다음 링크 참조](http://hbase.apache.org/book.html#basic.prerequisites)**
     <br><u>반드시 버전을 확인하고 사용</u>

## 3. Test environment
  |Java|Hadoop|HBase|Zookeeper|
  |:-:|:-:|:-:|:-:|
  |JDK8|2.7.7|1.4.10|HBase 내장|

## 4. Apache Hadoop
  **[> Download & Ref](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/SingleCluster.html)**

  ### 4.1. core-site.xml 수정
  ```xml
  <!-- $HADOOP_HOME/etc/hadoop/core-site.xml -->

  <!-- hdfs host:port 기입 -->
  <configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
  </configuration>
  ```

  ### 4.2. hadoop-env.cmd 수정
  ```cmd
  @rem $HADOOP_HOME/etc/hadoop/hadoop-env.cmd

  @rem JDK Path 기입
  set JAVA_HOME=C:\Users\DSS\jack\HBase\Java\jdk1.8.0_161
  ```

  ### 4.3. hdfs-site.xml 수정
  ```xml
  <!-- $HADOOP_HOME/etc/hadoop/hdfs-site.xml -->

  <!-- hdfs namenode/datanode path 설정 -->
  <configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///C:/Users/DSS/jack/HBase/data/hadoop/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///C:/Users/DSS/jack/HBase/data/hadoop/datanode</value>
    </property>
  </configuration>
  ```

## 5. Apache Zookeeper
  **[> Download](https://medium.com/@shaaslam/installing-apache-zookeeper-on-windows-45eda303e835)**
  
  * 본 문서에서는 HBase 내장 Zookeeper를 사용하였음.

## 6. Apache HBase
  **[> Download & Ref Guide](https://hbase.apache.org/book.html)**</br>

  ### 6.1. hbase-env.cmd 수정
  ```cmd
  @rem $HBASE_HOME/conf/hbase-env.cmd

  set JAVA_HOME=$JAVA_HOME
  ```

  ### 6.2. hbase-site.xml 수정
  ```xml
  <!-- $HBASE_HOME/conf/hbase-site.xml -->

  <!-- Hadoop에서 설정한 hdfs 사용 -->
  <!-- HBase 내장 znode dataDir 설정 -->
  <configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:9000/user/jack/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>false</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>C:/Users/DSS/jack/HBase/data/zookeeper</value>
    </property>
  </configuration>
  ```

## 7. Apache Phoenix
  **[> Download](https://phoenix.apache.org/download.html)**<br>
  **[> Ref Guide](https://phoenix.apache.org/installation.html)**
  
  ### 7-1. Copy & Paste
    1. $PHOENIX_HOME/*.jar -> $HBASE_HOME/lib/
    2. $HBASE_HOME/conf/hbase-site.xml -> $PHOENIX_HOME/bin/hbase-site.xml

  ### 7-2. Error catch
  ```py
  # $PHOENIX_HOME/bin/sqlline.py
  # line 108 수정

  # java_cmd = java + ' $PHOENIX_OPTS ' + \ ...... 
  java_cmd = java \ ......
  ```

## 8. Run 
  ### 8.1. Start hdfs
  ```bash
  # 1. Start ssh service 
  net start sshd
  # 2. namenode format
  $HADOOP_HOME/bin/hdfs namenode -format
  # 3. Start Namenode daemon & Datanode daemon
  $HADOOP_HOME/sbin/start-dfs.cmd
  ```
  * NameNode - http://localhost:50070/
  ### 8.2. Start HBase
  ```bash
  $HBASE_HOME/bin/start-hbase.cmd
  ```
  ### 8.3. Start Phoenix
  ```bash
  $PHOENIX_HOME/bin/sqlline.py localhost
  ```
  ### 8.4. Export HBase data on Phoenix
  ```bash
  !outputformat csv
  !record data.csv
  select * from TABLE_NAME
  !record
  ```

  ### 8.5. Error Report
  * Winutils.exe
  
    Windows 환경에서 발생하는 오류 해결을 위해 파일을 $HADOOP_HOME/bin에 추가해주어야 한다.<br> [관련 링크](https://arclab.tistory.com/121)  
  * Could not start ZK at requested port of 2181.

    HBase 내장 Zookeeper를 구동하기 때문에 가끔 2181이 아닌 다른 포트로 ZK가 실행되곤 한다.
    이때, Zoo Client 설정을 추가 해주어야 한다.
    ```xml
    <!-- $HBASE_HOME/conf/hbase-site.xml -->
    <property>
      <name>hbase.zookeeper.property.clientPort</name>
      <value>2182</value>
    </property>
    ```
  
## 9. Data Export & Import

```cli
bin/hbase org.apache.hadoop.hbase.mapreduce.Driver export [테이블명] /local/path

bin/hbase org.apache.hadoop.hbase.mapreduce.Driver import [테이블명] /local/path
```
  * **[More](http://www.rotanovs.com/hbase/hbase-importexport/)**

## 10. Table Viewer
  ### 1.0. HBase Web UI
  * HBase 자체 WebUI 제공.
    <br>standalone의 경우 16087 port. 상황에 따라 다를 수 있음
    <br>table description 확인가능.
  ### 1.1. Apache Ambari
  * Windows 지원 X
  ### 1.2. HBase Explorer
  * HBase 0.20.2만 가능
  ### 1.3. HBase Tool for Kakao
  * ~ HBase 1.2까지 지원
  ### 1.4. HAdmin
  * Standalone 시 Zookeeper 오류
  ### 1.5. Apache Zeppelin
  

## 11. <s>필요사항</s>
  * Table Viewer가 존재하는지 여부
  * Web에 뿌려지는 진동센서 Data의 TABLE, COLUMN 명, 대략적인 Data 양
  * HBase, Hadoop, Pheonix 각각의 Version
  * HBase, Pheonix가 실행되는 PC의 OS

## 12. Table 정보
  **테이블 명 : IIOT_TBL_TAG_DATA_HISTORY_DETAIL**

  |COLUMN|DESCRIPTION|
  |:-:|:-:|
  |TXN_TAG_ID|트랜잭션ID|
  |TAG_ID|TAG ID|
  |TENANT_ID|테넌트ID|
  |DEVICE_ID|디바이스ID|
  |TAG_VALUE|TAG 값|
  |TAG_VALUE_TXN_TIME|TAG History 처리시간|
  |||

  ```sql
  CREATE TABLE IF NOT EXISTS IIOT_TBL_TAG_DATA_HISTORY_DETAIL (
    TXN_TAG_ID CHAR(20) NOT NULL,
    TAG_ID VARCHAR NOT NULL,
    TENANT_ID VARCHAR,
    DEVICE_ID VARCHAR,
    TAG_VALUE VARCHAR,
    TAG_VALUE_TXN_TIME Date,
    CONSTRAINT my_pk PRIMARY KEY (TXN_TAG_ID, TAG_ID)
  );
  ```

----
**<center>2019-09-29 강재구</center>**
