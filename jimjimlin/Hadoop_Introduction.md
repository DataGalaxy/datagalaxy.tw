## Hadoop - 初見 ##

**MarkdownPad** is a full-featured Markdown editor for Windows.

這一章，將從Hadoop的基本概念開始講起，

Hadoop起源於Nutch，一個開放原始碼的網路搜尋引擎，為Lucene專案的一部分。

當時Nutch在開發網頁搜尋引擎時，面對網路擁有的天文資料量苦無對策。Google在2003-4年所發表的兩篇論文**GFS**、**MapReduce** 成為了Nutch的及時雨。

直至今日，Hadoop在資料界已成為了呼風喚雨的存在，核心仍是由**MapReduce**與**HDFS**(GFS→NDFS→HDFS)所組成。然而這對Hadoop來說，只是個狹義的定位。
	
Hadoop在廣義的架構下還包含了其他許多的相關子專案，如下

Common:一系列分散式的檔案系統和一般I/O的元件和介面(序列化、Java RPC、持久性資料結構)

Avro:一個序列化的系統，提供了高效的、跨語言PRC的持久性資料儲存

MapReduce:分散式資料處理模式和執行環境，可執行於大型商用機叢集

HDFS:分散式檔案系統，執行於大型商用機叢集

Pig:提供一個用以處理大量資料集合的Script語言**Pig Latin**，可讓使用者不懂Java也可執行MapReduce

Hive:一個分散式資料倉儲。提供類SQL指令，可轉化為MapReduce程序用在HDFS中

HBase:一個分散式的住列導向資料庫。使用了HDFS作為底層儲存，同時支援MapReduce的批次計算和點查詢

ZooKeeper:一個分散式的高可用性資料協調服務。提供了分散式鎖之類的服務，可應用於建置分散式應用

Sqoop:一個可在關聯式資料庫與HDFS之間傳輸資料的工具

在簡單介紹完這些相關子專案後，我們還是要拉回這一回的正題：介紹Hadoop基本概念

大家莫驚莫慌莫害怕，在未來，上面的相關技術DataGalaxy都會為大家深入的介紹

下面將在多說一點點MapReduce與HDFS後，結束這回合

### MapReduce ###

每個MapReduce工作分為**Map**及**Reduce**兩階段。兩階段皆使用<Key, Value>形式作為輸入與輸出。

Map的工作為將資料按照要求平行化，也就是所謂的**映射**

Reduce的工作為彙整Map程式所平行分散的資料，也就是所謂的**縮減**

在這邊直接用一個例子來解說：







### HDFS ###

HDFS是一個用以儲存大量資料流的檔案系統，擁有以下三個優點

Very large files:可支援Peta級的資料量

Streaming data access:

Commodity hardware:

神也是人，只是能作到人所不能做的事情

HDFS也存在著一些限制如下















