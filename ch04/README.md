# Chapter 04 -- 執行緒與並行性 (Threads & Concurrency) #

## Section Goals ##

* 確定執行緒的基本元素，並比較執行緒和行程
* 描述設計多執行緒行程的主要好處和重大挑戰
* 說明不同執行緒的處理方法，包括執行緒池、fork-join 和 Grand Central Dispatch
* 描述 Windows 和 Linux 作業系統的表示方式執行緒
* 使用 Pthreads、Java 和 Windows 設計多執行緒應用程式的執行緒 API

## Section ##

* [4.1 概論 (Overview)](#41-概論-overview)
* [4.2 多核心程式撰寫 (Multicore Programming)](#42-多核心程式撰寫-multicore-programming)
* [4.3 多執行緒模式 (Multithreading Models)](#43-多執行緒模式-multithreading-models)
* [4.4 執行緒程式庫 (Thread Libraries)](#44-執行緒程式庫-thread-libraries)
* [4.5 隱式執行緒 (Implicit Threading)](#45-隱式執行緒-implicit-threading)
* [4.6 執行緒的事項 ( Threading Issues)](#46-執行緒的事項--threading-issues)
* [4.7 作業系統範例 (Operating-System Examples)](#47-作業系統範例-operating-system-examples)
* [4.8 摘要 (Summary)](#48-摘要-summary)

## 4.1 概論 (Overview) ##

* thread 是 CPU 使用時的一個基本單位
* thread 是由一個 thread ID、程式計數器、一組 register 以及一個 stack 空間所組成。
* 在同一個 process 中，所有的 thread 共用程式碼區域、資料區域和作業系統資源。

<div style="text-align:center">
    <img src="../img/0401 - Single-threaded and multithreaded processes.png" alt= "0401 - Single-threaded and multithreaded processes" width="60%">
    <p>單執行緒行程與多執行緒行程</p>
</div>

### 4.1.1 動機 ###

* 許多在現代電腦和行動裝置執行的套裝軟體都是多執行緒。應用程式通常都製作成有許多執行緒控制的個別行程。
* process 的產生是費時且耗費資源的。
* 大部分 os kernal 是多執行緒。
* 許多應用程式可以利用多個執行緒，包括基本排序、樹狀和圖形演算法。

### 4.1.2 利益 ###

* 多執行緒程式的好處分成 4 個類別
  1. 應答 (Responsiveness) : 交談式的應用程式多執行緒化，在一部分被暫停或執行冗長的操作時，另一部分繼續執行。
  2. 資源分享 (Resource sharing) : process 只能由 shared-memory 和 message-passing 等技巧分享資源。而 threads 將共用它們所屬 process 的記憶體和資源。
  3. 經濟 (Economy) : thread 的產生和內文交換就比較經濟，且內文交換也比較快。
  4. 可擴展性 (Scalability) : 在多處理器的架構下，每一執行緒可以併行地在不同的處理核心上執行。

## 4.2 多核心程式撰寫 (Multicore Programming) ##

* 無論核心跨越 CPU 晶片或是在 CPU 晶片內，這種系統稱為多核心 (multicore) 系統。
* 並行 (concurrency) : 一個並行系統藉由 CPU 排成來允許所有的 task 有進展來支援一個以上的 task 。
* 平行 (parallelism) : 一個平行系統能同時執行一個以上的 task 。亦即，多個核心在同一時間分別去處理 task 。

### 4.2.1 程式撰寫的挑戰 ###

* 撰寫多核心系統的程式時，有 5 個領域面臨挑戰
  1. 辨識任務 (Identifying tasks) : 檢查應用程式來找出可以切割成獨立、並行的任務。
  2. 平衡 (Balance)
  3. 資料分割 (Data splitting)
  4. 資料相依 (Data dependency)
  5. 測試與偵錯 (Testing and debugging)

### 4.2.2 平行的類型 ###

* 兩種型態的平行
  1. 資料平行 (data parallelism) : 強調分配同一筆資料的子集合到多個運算核心，並在每一個核心執行相同的操作。 (分配資料到多個運算核心)
  2. 任務平行 (task parallelism) : 分配任務 (執行緒) 而非資料到多個運算核心。 (分配任務到多個運算核心)

<div style="text-align:center">
    <img src="../img/0405 - Data and task parallelism.png" alt= "0405 - Data and task parallelism" width="60%">
    <p>單執行緒行程與多執行緒行程</p>
</div>

## 4.3 多執行緒模式 (Multithreading Models) ##

## 4.4 執行緒程式庫 (Thread Libraries) ##

## 4.5 隱式執行緒 (Implicit Threading) ##

## 4.6 執行緒的事項 ( Threading Issues) ##

## 4.7 作業系統範例 (Operating-System Examples) ##

## 4.8 摘要 (Summary) ##
