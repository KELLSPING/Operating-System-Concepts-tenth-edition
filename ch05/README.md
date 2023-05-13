# Chapter 05 -- CPU 排班 (CPU Scheduling) #

* CPU scheduling 是多元程式 (multiprogramming) 規劃作業系統的基礎。
* 藉由 CPU 在不同 process 之間的轉換，OS 可以讓電腦的產量提高。
* 當討論一般的 scheduling 概念時，使用的是行程排班 (process scheduling) ；而提到特定執行緒觀念時，使用的是執行緒排班 (thread scheduling) 。
* 核心 (core) 是 CPU 的基本單位。行程 (process) 是在 CPU 的核心上執行的。
* 使用將行程排程為 "在 CPU 上執行" 的通用術語，就意味著該行程在 CPU 的核心上執行。

## Section Goals ##

* Describe various CPU scheduling algorithms.
  * 描述各種 CPU 排班演算法
* Assess CPU scheduling algorithms based on scheduling criteria.
  * 根據排班標準評估 CPU 排班演算法
* Explain the issues related to multiprocessor and multicore scheduling.
  * 解釋多處理器和多核心排班相關的問題
* Describe various real-time scheduling algorithms.
  * 描述各種即時排班演算法
* Describe the scheduling algorithms used in the Windows, Linux, and
Solaris operating systems.
  * 描述 Windows、Linux 和 Solaris 作業系統中使用的排班演算法
* Apply modeling and simulations to evaluate CPU scheduling algorithms.
  * 應用模型和模擬來評估 CPU 排班演算法
* Design a program that implements several different CPU scheduling algorithms.
  * 設計一個流程實現幾種不同的 CPU 排班演算法

## Section ##

* [5.1 基本觀念 (Basic Concepts)](#51-基本觀念-basic-concepts)
* [5.2 排班原則 (Scheduling Criteria)](#52-排班原則-scheduling-criteria)
* [5.3 排班演算法 (Scheduling Algorithms)](#53-排班演算法-scheduling-algorithms)
* [5.4 執行緒排班 (Thread Scheduling)](#54-執行緒排班-thread-scheduling)
* [5.5 多處理器的排班問題 (Multi-Processor Scheduling)](#55-多處理器的排班問題-multi-processor-scheduling)
* [5.6 即時 CPU 排班 (Real-Time CPU Scheduling)](#56-即時-cpu-排班-real-time-cpu-scheduling)
* [5.7 作業系統範例 (Operating-System Examples)](#57-作業系統範例-operating-system-examples)
* [5.8 演算法的評估 (Algorithm Evaluation)](#58-演算法的評估-algorithm-evaluation)
* [5.9 摘要 (Summary)](#59-摘要-summary)

## 5.1 基本觀念 (Basic Concepts) ##

* 在單一 CPU 核心系統裡，同一時間只能有一個行程在執行，等 CPU 的核心有空時，才能接著重新排班。
* 多元程式 (multiprogramming)
  * 將多的 process 同時存放在記憶體中。
  * 隨時保有一個行程在執行，藉以提高 CPU 使用率。
* 幾乎所有的計算機資源 (ex. CPU) 都在使用之前先排班。

### 5.1.1 CPU-I/O 分割週期 (CPU-I/O Burst Cycle) ###

* Process 的執行是由 CPU 執行時間及 I/O 等待時間所組成的週期 (cycle) ，process 在這兩個狀態之間交替往返。
* Process 執行由一個 CPU 分割 (CPU burst) 開始，接著是一個 I/O 分割 (I/O burst) ，然後又是一個 CPU 分割，再接著一個 I/O 分割，依此方式繼續。
* 最後一個 CPU 分割結束的時候，同時會有一個 system request 中止執行這個工作。

<div style="text-align:center">
    <img src="../img/0501 - Alternating sequence of CPU and I_O bursts.png" alt= "0501 - Alternating sequence of CPU and I_O bursts.png" width="30%">
    <p>CPU 分割和 I/O 分割交替排列的順序</p>
</div>

### 5.1.2 CPU 排班器 (CPU Scheduler) ###

* 當 CPU idle 時，OS 必須從就緒佇列 (ready queue) 之中選出一個 process 來執行。
* 選取 process 是由 CPU 排班器 (CPU schedular) 來執行，schedular 自記憶體之中準備要執行的數個 process 中選出一個，並將 CPU 配置給它。
* ready queue 可製作成 FIFO queue 、priority queue 、tree queue 或僅為毫無順序的 linked list 。
* queues 中的紀錄通常是 process 的行程控制表 (process control block, PCBs) 。

### 5.1.3 可搶先與不可搶先排班 (Preemptive and Nonpreemptive Scheduling) ###

* CPU scheduling 的決策發生在下面的 4 種狀況
  1. Process 從 running 轉變成 waiting (ex. I/O 要求、呼叫 wait() 等待子行程的結束)
  2. Process 從 running 轉變成 ready (ex. 當有 interrupt 發生時)
  3. Process 從 waiting 轉變成 ready (ex. I/O 的結束)
  4. Process terminated

<div style="text-align:center">
    <img src="../img/0302-diagram_of_process_state_2.png" alt= "0302-diagram_of_process_state_2.png" width="60%">
    <p>行程狀態圖</p>
</div>

* 對狀況 1 和 4 而言，CPU 只能在 ready queue 中選擇一個新的 process 來執行。
* 如果 scheduling 只發生在 1 和 4 時，我們稱這種排班方法為不可搶先 (nonpreemptive) 或合作 (cooperative) ；否則就稱為可搶先 (preemptive) 。
  * nonpreemptive, cooperative : I/O 要求、呼叫 wait() 等待子行程的結束
  * preemptive : interrupt 、I/O 的結束
* 在 cooperative 方法下，一旦 CPU 配置給一個行程時，此行程將一直保有 CPU ，直到它 terminated 或轉換到 waiting 狀態，釋放出 CPU 為止。
* 幾乎所有現在 OS 都使用 preemptive scheduling algorithms 。 (ex. Windows, macOS, Linux, UNIX)
* 可搶先排班法在存取共用資料時，可能造成競爭關係 (race condition) 。
* OS kernal 可以設計為不可搶先或可搶先。
  * nonpreemptive kernal 將在 system call 完成或等待行程阻塞發生時 I/O 完成，才作 content switch ，以便解決 race condition 。
  * preemptive kernal 需要諸如 mutex lock 之類的機制，防止在存取共享核心資料結構時出現 race condition 。
* nonpreemptive 的執行模式對於支援任務必須在一定時間內完成的即時計算是比較差的，在 5.6 節會探討即時系統 (real-time systems) 的排班需求。
* 大多數現代 OS 在以核心模式 (kernal mode) 運行時，都完全可搶先。

### 5.1.4 分派器 (Dispatcher) ###

* 在 CPU scheduling 功能包含的另外一個元件就是分派器 (dispatcher)，dispatcher 是一個模組，它將 CPU core 的控制權交給 CPU scheduler 選出的行程。
* 分派器的功能包括
  1. 內容轉換 (content switch)
  2. 轉換成使用者模式 (user mode)
  3. 跳躍到使用者程式的適當位置，以便重新開啟程式
* 分派器用來停止一個行程，並啟動另一個行程所用的時間，就是所謂的分派延遲 (dispatch latency) 。

<div style="text-align:center">
    <img src="../img/0503 - The role of the dispatcher.png" alt= "0503 - The role of the dispatcher.png" width="25%">
    <p>分派器的腳色</p>
</div>
  
## 5.2 排班原則 (Scheduling Criteria) ##

* 不同的 CPU-scheduling 方法有不同的特性，選擇某一演算法會對某類行程比較有利。
* 有多種評定的標準可以用來作為 CPU 排班法則的評估參考，其中不同標準決定出來的最佳演算法也各不相同，通常有以下幾種標準:
  1. CPU 使用率 (CPU utilization) : 在實際系統中，CPU 使用率應該是介於 40% (負荷較輕) 到 90% (負荷較重) 。
  2. 傳輸量 (throughput) : 每個單位時間所完成的行程數。
  3. 回復時間 (Turnaround time) : 行程需要多少時間完成。
  4. 等待時間 (Waiting time) : 在就緒佇列中等待花費周期的總合。
  5. 回應時間 (Response time) : 以提出一個要求到第一個回應出現的時間間隔來計算。是指開始有所回應的時間，而不適只完成回應的時間。
* 希望最大化 CPU utilization 和 throughput ，並且降低 Turnaround time 、Waiting time 和 Response time 。在大多數情況下，是將平均值最佳化。如果為了保證每一個使用者都得到好的服務，則可能要求它的最長回復時間減為最小化。在交談式系統中，最重要的是能讓 Response time 的差異達到最小。

## 5.3 排班演算法 (Scheduling Algorithms) ##

* CPU-scheduling 排班所處理的問題是，如何決定將 CPU core 分配給 ready queue 中的哪一個行程
* 儘管大多數現在 CPU 架構都具有 multi-cores ，但我們僅在 single-core 中描述這些 CPU-scheduling algorithms
* single CPU 且只有一個 CPU core ，因此系統一次只能執行一個 process

### 5.3.1 先來先做排班法 (first-come, first-served; FCFS) ###

* 目前最簡單的 CPU 排班演算法，就是 FCFS 排班演算法，就是把 CPU 資源分配給第一個要求 CPU 的行程。
* FCFS 策略的製作很容易用 FIFO 佇列管理，當一個 process 進入 ready queue 後，它的行程控制區段 (PCB) 就鏈結到串列的尾端。當 CPU 有空時，就分配資源給在就緒佇列開頭的行程，執行中的行程就會從就緒佇列中剃除。
* 缺點 : FCFS 方法下的平均等待時間經常是很長的。
* FCFS 排班演算法是 nonpreemptive 的演算法。

### 5.3.2 最短的工作先做排班法 (shortest-job-first, SJF) ###

* 將每一個行程的下一個 CPU 分割長度和該行程相結合，當 CPU 有空時，就指定給下一個 CPU 分割最短的行程。如果兩個行程具有相同長度的下一個 CPU 分割，就採用先來先做 (FCFS) 方法。
* 更適當的說法 : 最短的下一個 CPU 分割 (shortest-next-CPU-burst) 排班演算法。
* SJF 演算法可以是不可搶先或可搶先的。如果一個行程正在執行，而另有一個新行程到達就緒佇列中，就會產生問題。
* 可搶先的 SJF 排班有時候又稱為最短剩餘時間優先 (shortest-remaining-time-first) 。

### 5.3.3 依序循環排班法 (round-robin, RR) ###

* 和 FCFS 排班法相類似，但是加入可搶先的規則，讓行程互相交換使用 CPU。
* 定義一個小的時間單位，稱為一個時間量 (time quantum) 或是時間片段 (time slice) 。通常一個時間量是 10 ~ 100 ms 。
* 就緒佇列視為一個環狀佇列，CPU 排班程式繞著這個就緒佇列走，分配 CPU 給每一個行程一個時間量的時間區段。
* 為了製作 RR 排班法，將就緒佇列當成行程的 FIFO 佇列，新的行程就加到就緒佇列的尾端。CPU 排班程式從就緒佇列中挑出第一個行程，設定計時器在經過一個時間量之後會發出中斷信號，並且分派該行程。
* 接下來有兩種情況可能發生
  1. 行程的 CPU 分割可能比一個時間量小，在這樣的情況下，行程本身自動交還 CPU，於是排班器可以繼續進行在就緒佇列中的下一個行程。
  2. 如果目前正在執行的 CPU 分割比一個時間量長，計時器將停止，並對作業系統產生一個中斷。執行 content switch ，且將該行程至於就緒佇列的尾端。 CPU 排班程式接著在就緒佇列中選出下一個行程。
* 在 RR 方法下的平均等待時間通常較長。
* RR 排班將產生大量的內容轉換。

## 5.4 執行緒排班 (Thread Scheduling) ##

## 5.5 多處理器的排班問題 (Multi-Processor Scheduling) ##

## 5.6 即時 CPU 排班 (Real-Time CPU Scheduling) ##

## 5.7 作業系統範例 (Operating-System Examples) ##

## 5.8 演算法的評估 (Algorithm Evaluation) ##

## 5.9 摘要 (Summary) ##
