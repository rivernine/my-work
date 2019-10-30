# HDFS + H2O Driverless AI

## 0. 개요
HDFS data를 H2O Driverless AI에 활용하고자 한다.<br>
(Local 환경 기준으로 기술)

* HDFS: Local PC (Windows 10)
* H2O Driverless AI: Docker Container

## 1. Environment
  |Tools|Version|
  |:-:|:-:|
  |Hadoop|2.7.7|
  |H2O|Driverless AI 1.7.0|

## 2. Hadoop
  * **[> Download & Ref](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/SingleCluster.html)**
  * Test를 위하여 Standalone으로 구성.

  ### 2.1. core-site.xml 수정
  ```xml
  <!-- $HADOOP_HOME/etc/hadoop/core-site.xml -->

  <!-- hdfs host:port 기입 -->
  <configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://0.0.0.0:9000</value>
    </property>
  </configuration>
  ```

  ### 2.2. hadoop-env.cmd 수정
  ```cmd
  @rem $HADOOP_HOME/etc/hadoop/hadoop-env.cmd

  @rem JDK Path 기입
  set JAVA_HOME=C:\Users\DSS\jack\HBase\Java\jdk1.8.0_161
  ```

  ### 2.3. hdfs-site.xml 수정
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

## 3. H2O Driverless AI (Docker)
  * AutoML on H2O 
  * **[> How to Install](https://www.h2o.ai/products/h2o-driverless-ai/)**
  * **[> Document](http://docs.h2o.ai/driverless-ai/latest-stable/docs/userguide/index.html)**

  ### 3.1. Supported Hadoop Platforms
  * HDP 2.2
  * HDP 2.3
  * HDP 2.4
  * HDP 2.5
  * HDP 2.6
  * [More](http://docs.h2o.ai/driverless-ai/latest-stable/docs/userguide/connectors/hdfs.html)

  ### 3.2. License
  * H2O Driverless AI에서 21일간 무료 License를 제공한다.
    License key를 메일로 받고, $H2O_HOME/license/license.sig로 저장한다

  ### 3.3. Create run script


  
  ```cmd
  @rem * run.cmd 생성 후 script작성
  docker run \
  --name "h2o" \
  --init \
  --rm \
  --shm-size=256m \
  -p 12345:12345 \
  -p 9000:9000 \
  -p 50010:50010 \
  -p 8020:8020 \
  -e DRIVERLESS_AI_CONFIG_FILE="/config/config.toml" \
  -e DRIVERLESS_AI_ENABLED_FILE_SYSTEMS="file,hdfs" \
  -e DRIVERLESS_AI_LOCAL_AUTHENTICATION_METHOD="none" \
  -e DRIVERLESS_AI_PROCSY_PORT=8080 \
  -v C:/Users/DSS/jack/H2O/dai_rel_1.7.0/data:/data \
  -v C:/Users/DSS/jack/H2O/dai_rel_1.7.0/log:/log \
  -v C:/Users/DSS/jack/H2O/dai_rel_1.7.0/license:/license \
  -v C:/Users/DSS/jack/H2O/dai_rel_1.7.0/tmp:/tmp \
  -v C:/Users/DSS/jack/H2O/dai_rel_1.7.0/config:/config \
  h2oai/dai-centos7-x86_64:1.7.0-cuda10.0
  ```

## 4. Run
### 4.1. Start Hadoop
  ```bash
  $HADOOP_HOME/sbin/start-all.cmd
  ```

### 4.2. Start H2O Driverless AI (Docker)
  ```bash
  $H2O_HOME/run.cmd
  ```

### 4.3. Add Dataset (HDFS)
  * 4.3.1. Go to localhost:12345
  * 4.3.2. Click ADD DATASET and HADDOP FILE SYSTEM
  * 4.3.3. Connect HDFS
    - **Docker container 내부(H2O)와 Host(HDFS)의 통신을 위해 host.docker.internal를 이용**
      e.g. hdfs://host.docker.internal:9000/user/hive/warehousd/test.db

## 5. 검토
### 5.1. 
  * 외부 HDFS로 접근은 권한이 필요할 것으로 생각됨.
  * H2O Driverless AI가 GPU를 지원하긴하나 Docker Container로 가동할 경우,
    Host와 Container의 OS가 동일해야함.
    실제 설치 시 H2O Driverless AI Native/Container 중 선택하여 설치.

---
**<center>2019-09-27 강재구</center>**
