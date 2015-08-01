# Hadoop cluster 安裝

## 介紹
本篇文章主要介紹如何建置一個 Hadoop cluster 基本操作環境，在進入如何安裝 Hadoop cluster 安裝之前，先簡單說明 Hadoop 的架構、運作方式，因為先了解這些之後，在後面安裝、設定時，會更清楚為何要做這些設定，所以建議可以先了解基本的 Hadoop 架構，再進入安裝、設定會更清楚。

### HDFS

在 Hadoop 中最核心的部分為 Hadoop Distributed File System (HDFS)，是一個分散式的檔案系統，能提供可擴展、信賴的檔案系統，主要是負責將資料分散的儲存到不同的節點主機上，並且能避免某一節點主機失效造成資料遺失的情況。

以本篇文章實作來說，HDFS 由 1 個 NameNode 主機與多個 DataNodes 主機組成，NameNode 負責管理 HDFS 中的 metadata，如檔案的權限、建立日期等，而 DataNodes 提供資料儲存。

以圖 1 所示，有一份檔案儲存在 HDFS 中，NameNode 會把這一份檔案分成多個 blocks，分散儲存在 DataNodes 中，所以，NameNode 需要紀錄哪一個檔案的 block 儲存在哪些 DataNodes 中，此外，為了避免單一個 DataNode 損壞造成資料遺失，NameNode 還需要將 blocks 的複本儲存到不同的 DataNodes 中做備份。

// 圖

### MapReduce
MapReduce 能提供處理大量資料的軟體框架，分別藉由 *Map* 與 *Reduce* 兩個 procedures 達成。*Map* 能把大量的資料區分成多個獨立的資料集，送到 cluster 中平行執行，*Reduce* 是利用 *Map* procedure 產生排序、過濾後的輸出進行加總產生出最後的結果。

在 Hadoop 0.23 之後的版本都是使用 **MapReduce 2.0** (簡稱 **YARN** 或 **MRv**2)，YARN 修改了之前 MapReduce 的架構。

在下面說明 YARN 的架構：

// YARN

## 實作環境

* CentOS 6.6 x64
* Java 1.7.0_79
* Hadoop 2.7.1

### Cluster 環境

* 總共有 3 個節點 (nodes)
* 1 個 master (NameNode, DataNode)：hadoop01
* 2 個 slaves (DataNodes)：hadoop02, hadoop03

## 準備安裝
這次安裝實作的作業系統以 CentOS 6.6 為主，在進入安裝之前，有一些前置步驟需要先完成，在之後進行安裝、設定 Hadoop cluster 時候會降低遇到一些問題。

#### 校時
因為 Hadoop cluster 是由多個節點 (nodes) 組成的分散式環境，所以需要同步每個 node 的系統時間，能直接與網路上的 NTP server 同步系統時間，或者把某一個節點作為 NTP server，其他的節點都向 cluster 內的 NTP server 進行校時。本篇文章並不著重在 NTP 校時的設定上，所以就不對其細節進行說明。

#### 防火牆
Hadoop cluster 中，每個節點都是透過網路連接，會使用到特定的 ports，如果不確定會使用哪些 ports，可以暫時把每個節點主機的 *iptables*、其他防火牆關閉，或者設定允許所有節點的 IP 位址、網段連線，以下對 iptables 的操作進行說明。

##### 方法 1. 關閉 iptables
```
[datagalaxy@hadoop01 ~]$ sudo service iptables stop
[datagalaxy@hadoop01 ~]$ sudo chkconfig iptables off
```

##### 方法 2. 允許特定 IP 位址或網段
```
[datagalaxy@hadoop01 ~]$ sudo iptables -I INPUT 1 -s XXX.XXX.XXX.XXX -j ACCEPT
[datagalaxy@hadoop01 ~]$ sudo iptables -L 
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  XXX.XXX.XXX.XXX      anywhere
// ignore

[datagalaxy@hadoop01 ~]$ sudo iptables-save > /etc/sysconfig/iptables
[datagalaxy@hadoop01 ~]$ sudo servie iptables restart
```

請將上面的 `XXX.XXX.XXX.XXX` 替換成自己的 IP 位址

#### 關閉 SELinux

##### 方法 1. 暫時關閉 SELinux
```
[datagalaxy@hadoop01 ~]$ sudo setenforce 0
```
注意：重開機之後就失效

##### 方法 2. 永久關閉 SELinux
```
[datagalaxy@hadoop01 ~]$ sudo vi /etc/selinux/config
```

修改下面內容，將 `SELINUX` 的設定值改成：

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled

# ignore
```

#### 設定 /etc/hosts

如果不想直接使用 IP 位址與節點主機連線，可以在每台的 `/etc/hosts` 加入主機名稱與 IP 位址，方便使用主機名稱就能與節點主機連線。

在下面的說明都以主機名稱來表示：

* **master node:** hadoop01
* **slave nodes:** hadoop02, hadoop03

```
[datagalaxy@hadoop01 ~]$ vi /etc/hosts
```

加入以下內容：

```
192.168.1.1    hadoop01
192.168.1.2    hadoop02
192.168.1.3    hadoop03
```

#### SSH keys
在 Hadoop 中，每個 node 需要利用 SSH 連線，所以需要使用 SSH keys 來進行登入驗證。

##### 1. 分別在 master, slaves 建立 hadoop 帳號

```
[datagalaxy@hadoop01 ~]$ sudo adduser hadoop
[datagalaxy@hadoop01 ~]$ sudo passwd hadoop
```

##### 2. 產生 hadoop 帳號的 public 及 private keys

```
[datagalaxy@hadoop01 ~]$ su hadoop
[hadoop@hadoop01 datagalaxy]$ cd ~
[hadoop@hadoop01 ~]$ ssh-keygen
```

##### 3. 把 public key 加入到 authorized_keys

```
[hadoop@hadoop01 ~]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[hadoop@hadoop01 ~]$ chmod 600 ~/.ssh/authorized_keys
```

##### 4. 複製 public, private keys 及 authorized_keys 到所有節點主機

```
[hadoop@hadoop01 ~]$ scp ~/.ssh/authorized_keys hadoop@hadoop02:~/.ssh/authorized_keys
[hadoop@hadoop01 ~]$ scp ~/.ssh/authorized_keys hadoop@hadoop03:~/.ssh/authorized_keys
```

上面步驟需要輸入密碼登入

```
[hadoop@hadoop01 ~]$ scp ~/.ssh/id_rsa.pub hadoop@hadoop02:~/.ssh/id_rsa.pub
[hadoop@hadoop01 ~]$ scp ~/.ssh/id_rsa.pub hadoop@hadoop03:~/.ssh/id_rsa.pub

[hadoop@hadoop01 ~]$ scp ~/.ssh/id_rsa hadoop@hadoop02:~/.ssh/id_rsa
[hadoop@hadoop01 ~]$ scp ~/.ssh/id_rsa hadoop@hadoop03:~/.ssh/id_rsa
```

##### 5. 測試是否能不能使用 SSH keys 直接與其他節點主機連線

```
[hadoop@hadoop01 ~]$ ssh hadoop02
```

## 安裝與設定

Apache Hadoop 是由 Java 語言開發，所以主機節點需安裝 Java，Hadoop 提供 binary 與 source 兩種檔案，如果不想自己編譯 source code，可以直接下載 binary，本次實作也是以 binary 為例。

Apache Hadoop 下載網址：[http://hadoop.apache.org/releases.html]()

### 安裝 JDK
主要有兩種安裝方式：

#### 1. Oracle Java

至 Oracle Java 取得下載網址：[http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html]()

```
[hadoop@hadoop01 ~]$ wget http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.rpm
[hadoop@hadoop01 ~]$ sudo rpm -ivh jdk-7u79-linux-x64.rpm
```

#### 2. OpenJDK

```
[hadoop@hadoop01 ~]$ sudo yum install java-1.7.0-openjdk-devel
```

#### 測試 Java

```
[hadoop@hadoop01 ~]$ java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

### 下載 Apache Hadoop Binary 檔案

```
[hadoop@hadoop01 ~]$ wget http://ftp.mirror.tw/pub/apache/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz
[hadoop@hadoop01 ~]$ tar -xvf hadoop-2.7.1.tar.gz
[hadoop@hadoop01 ~]$ mv hadoop-2.7.1. hadoop
```

### 設定 Hadoop

Apache Hadoop 的設定檔位於解開 tar 中目錄的 `etc/hadoop/`，以這次實作來說，設定檔路徑位於 `/home/hadoop/hadoop/etc/hadoop/` 內。

針對下面主要會修改到的設定檔進行說明：

* **core-site.xml** : 設定 HDFS 中 NameNode URI
* **hdfs-site.xml** : HDFS NameNode 與 DataNode 的其他設定
* **yarn-site.xml** : YARN 架構中的 components，如 ResourceManager、NodeManager 等及其他參數設定
* **mapred-site.xml** : MapReduce 的參數設定


以下的設定檔先修改 master node 中的設定，修改完之後再複製到其他 slave nodes 中。

#### 1. 加入 NameNode URI (core-site.xml)

```
[hadoop@hadoop01 ~]$ cd ~/hadoop/etc/hadoop
[hadoop@hadoop01 ~]$ vi core-site.xml
```

在 `core-site.xml` 中加入設定如下：

```
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop01:9000/</value>
  </property>
</configuration>
```

##### 參數說明：

* `fs.defaultFS` : 設定 NameNode URI，其中 `hadoop01` 為 NameNode 的主機名稱 

#### 2. 設定 HDFS (hdfs-site.xml)

在 `etc/hadoop/hdfs-site.xml` 中設定 HDFS 的參數，包括 NameNode 與 DataNode 的儲存路徑等

在 `hdfs-site.xml` 中加入以下設定：

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/home/hadoop/datanode</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/home/hadoop/namenode</value>
  </property>
</configuration>
```

##### 參數說明：

* `dfs.replication`: block 備份的數量
* `dfs.datanode.data.dir`: DataNode 儲存 block 的路徑
* `dfs.namenode.name.dir`: NameNode 儲存 name table 的路徑

#### 設定 YARN (yarn-site.xml)


### 更多關於 Hadoop 的設定

## 啟動 Hadoop

### 測試

## 關閉 Hadoop

## 常見問題
##### 1. 執行程式時常會噴出 xxx
##### 2. 執行失敗該怎麼除錯
