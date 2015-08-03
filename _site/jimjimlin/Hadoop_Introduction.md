## Hadoop - 初見 ##

>- 數大便是美。數大了似乎按照著一種自然律，自然的會有一種特別的排列，一種特別的節奏，一種特殊的式樣，激動我們審美的本能，激發我們審美的情緒。  *~徐志摩日記西湖記*

這一章，是 Hadoop 的第一章，將從 Hadoop 的基本概念開始講起

Hadoop 起源於 Nutch，一個開放原始碼的網路搜尋引擎，為 Lucene 專案的一部分

當時 Nutch 在開發網頁搜尋引擎時，面對網路擁有的天文資料量苦無對策。 Google 在2003-4年所發表的兩篇論文 **GFS**、 **MapReduce**  成為了 Nutch 的及時雨。

直至今日，Hadoop 在資料界已成為了呼風喚雨的存在，核心仍是由 **MapReduce** 與 **HDFS**(GFS→NDFS→HDFS) 所組成。然而這對 Hadoop 來說，只是個狹義的定位。
	
Hadoop 在廣義的架構下還包含了其他許多的相關子專案，如下

Common: 一系列分散式的檔案系統和一般I/O的元件和介面(序列化、Java RPC、持久性資料結構)

Avro: 一個序列化的系統，提供了高效的、跨語言PRC的持久性資料儲存

MapReduce: 分散式資料處理模式和執行環境，可執行於大型商用機叢集

HDFS: 分散式檔案系統，執行於大型商用機叢集

Pig: 提供一個用以處理大量資料集合的 Script語言 **Pig Latin** ，可讓使用者不懂 Java 也可執行 MapReduce

Hive: 一個分散式資料倉儲。提供類SQL 指令，可轉化為 MapReduce 程序用在 HDFS 中

HBase:一個分散式的住列導向資料庫。使用了HDFS作為底層儲存，同時支援 MapReduce 的批次計算和點查詢

ZooKeeper: 一個分散式的高可用性資料協調服務。提供了分散式鎖之類的服務，可應用於建置分散式應用

Sqoop: 一個可在關聯式資料庫與 HDFS 之間傳輸資料的工具

在簡單介紹完這些相關子專案後，我們還是要拉回這一回的正題：介紹 Hadoop 基本概念

大家莫驚莫慌莫害怕，在未來，上面的相關技術 DataGalaxy 都會為大家深入的介紹

下面將在多說一點點 MapReduce 與 HDFS 後，結束這回合

### MapReduce ###

每個 MapReduce 工作分為 **Map** 及 **Reduce** 兩階段。兩階段皆使用 <Key, Value> 形式作為輸入與輸出。

Map的工作為將資料按照要求平行化，也就是所謂的**映射**

Reduce的工作為彙整Map程式所平行分散的資料，也就是所謂的**縮減**

在這邊直接用一個例子來解說：

凱文是一隻雞，最近在Open農場實習，他的上司是一隻獅子，叫做阿銘

阿銘交待凱文負責統籌公司內部所有員工對於外送飲料廠商的意見

凱文接洽了三間飲料公司，分別是：威姐、GOCO、臉很臭紅茶店

然而，Open農場實在是太大了，凱文沒辦法自己親自一個一個詢問員工的意見

凱文在請教了強森同事後，終於得到了解決辦法

凱文假傳聖旨讓其他實習生們各自負責一畝田地

實習生們在各自負責的田地中實行意見調查後，在回報給凱文

凱文最後統一彙整後，決定了日後合作的飲料廠商為GOCO後回報給阿銘，輕鬆解決了這次的任務。

在這個故事中，凱文將一樣的工作分給每一畝地的這個動作即為 **Map**

而凱文最後統一彙整的動作則為 **Reduce**

用這個例子應該可以讓大家多少體會一下 MapReduce 的精隨了

在將來的章節裡面，我們也會帶領大家動手實作，屆時請在好好的體會 MapReduce的奧妙 

### HDFS ###

HDFS 是一個用以儲存大量資料流的檔案系統，擁有以下三個優點

Very large files: 可支援Peta級的資料量

Streaming data access: HDFS使用的是讀取一次，多次讀取的資料儲存技術，相對於讀取單位為 packet 或 chunk 的資料，HDFS 讀取資料的方式的是連續的恆定 bitrate，非常適合資料的批次處理

Commodity hardware: Hadoop不需要昂貴的成本，一般的硬體即可做平行擴充，用以儲存大量的資料

*神也是人，只是能做到人所不能做的事情*

HDFS 也存在著一些限制如下

Low-latency data access: 在Hadoop應用中，有不少工作如: 資源分配、Map function、、、等造成延遲性較高，目前 HBase 在這方面表現得較好

Lots of small files: Namenode 掌握了整個檔案系統的 metadata，因此要是把資料分成太多微小資料檔，會耗費太多空間存放 metadata

Multiple writers, arbitrary file modifications: 目前的 HDFS 還不支援 multiple writers，一次只存在一個writer

HDFS 的簡單概念介紹就先到這裡，往後將會透過實作配合概念，讓大家逐步了解整個系統的核心

最後，請大家繼續支持 DataGalaxy，學習更多資料科學知識


下集預告: Hadoop Cluster 的環境設定安裝










