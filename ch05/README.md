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
* FCFS 策略的製作很容易用 FIFO 佇列管理，當一個 process 進入 ready queue 後，它的行程控制區段 (PCB) 就鏈結到串列的尾端。當 CPU 有空時，就分配資源給在 ready queue 開頭的行程，執行中的行程就會從 ready queue 中剃除。
* 缺點 : FCFS 方法下的平均等待時間經常是很長的。
* FCFS 排班演算法是 nonpreemptive 的演算法。

### 5.3.2 最短的工作先做排班法 (shortest-job-first, SJF) ###

* 將每一個行程的下一個 CPU 分割長度和該行程相結合，當 CPU 有空時，就指定給下一個 CPU 分割最短的行程。如果兩個行程具有相同長度的下一個 CPU 分割，就採用先來先做 (FCFS) 方法。
* 更適當的說法 : 最短的下一個 CPU 分割 (shortest-next-CPU-burst) 排班演算法。
* SJF 演算法可以是不可搶先或可搶先的。如果一個行程正在執行，而另有一個新行程到達就緒佇列中，就會產生問題。
* preemptive 的 SJF 排班有時候又稱為最短剩餘時間優先 (shortest-remaining-time-first) 。

### 5.3.3 依序循環排班法 (round-robin, RR) ###

* 和 FCFS 排班法相類似，但是加入 preemptive 的規則，讓行程互相交換使用 CPU。
* 定義一個小的時間單位，稱為一個時間量 (time quantum) 或是時間片段 (time slice) 。通常一個時間量是 10 ~ 100 ms 。
* ready queue 視為一個 circular queue ，CPU 排班程式繞著這個就緒佇列走，分配 CPU 給每一個行程一個時間量的時間區段。
* 為了製作 RR 排班法，將就緒佇列當成行程的 FIFO 佇列，新的行程就加到 ready queue 的尾端。CPU 排班程式從 ready queue 中挑出第一個行程，設定 timer 在經過一個時間量之後會發出 interrupt ，並且分派該行程。
* 接下來有兩種情況可能發生
  1. process 的 CPU burst 可能比一個時間量小，在這樣的情況下，行程本身自動交還 CPU，於是 scheduler 可以繼續進行在 ready queue 中的下一個行程。
  2. 如果目前正在執行 process 的 CPU burst 比一個時間量長，timer 將停止，並對 OS 產生一個 interrupt 。執行 content switch ，且將該行程至於 ready queue 的尾端。 CPU scheduler 接著在 ready queue 中選出下一個行程。
* 在 RR 方法下的平均 waiting time 通常較長。
* RR 排班將產生大量的 content switch 。
* 因此 RR scheduling 中，沒有一個 process 所分配的 CPU core 時間會超過 1 time quantum ，除非只有一個可執行程式。如果 process 的 CPU burst 超過 1 time quantum ，它將被搶先，並且放回 ready queue 中。 RR scheduling 是一種 preemptive 的排班演算法。

### 5.3.4 優先權排班法 (priority-scheduling) ###

* 最短的工作先做排班法 (SJF) 是一般優先權排班演算法的特例。
* 將 CPU core 分配給具有最高優先權的過程。 優先權相同的 process 按 FCFS 順序安排。
* 優先權一般是一些固定範圍的數字 (ex. 0~7, 0~4096)。有些系統使用低數值表示高優先權，有些則相反。
* 優先權可以由內部或外部定義。
  * 內部得到的優先權是使用一些可以測得的量 (ex. 時間限制、記憶體需求、開啟檔案數量、平均 I/O
  burst 與平均 CPU burst 的比率)
  * 外部得到的優先權是由作業系統外部的一些標準所決定 (ex. 行程的重要性、支付使用電腦所付經費的類型與數量、其他政策性的因素)
* 優先權可以是 preemptive 或 nonpreemptive 的。
  * preemptive : 如果新 process 到達 ready queue 後，它的優先權如果比目前執行中 process 的優先權高，就會搶走 CPU core 先做。
  * nonpreemptive : 將新的 process 放在 ready queue 的前端。
* 無限期阻塞 (indefinite blocking) 問題、 飢餓 (starvation) 問題
  * priority-scheduling 可能會造成一些低優先權的 process 一直等待。如果在一個工作繁重的電腦系統中，一連串高優先權的 process 會讓低優先權的 process 始終得不到 CPU。
  * 低優先權的 process 遭遇無限期阻塞的一種解決方法，採用老化 (aging)。
* 老化 (aging)
  * 逐漸提高停留在系統中已經過一段長時間的 process 之優先權。
  * 如果優先權範圍是 0~127 ，可以定期 (ex. 每秒) 遞增某個 process 的優先權，即是將優先權範圍的數字減一。
* 依序循環排班 (RR) + 優先權排班 (priority)
  * 使系統執行最高優先權 process ，並使用 RR 來執行具有相同優先權的 process 。
  * 所有 process 都放在單一佇列中，可能需要 O(n)。

### 5.3.5 多層佇列排班法 (multi-level queue scheduling) ###

* 使用依序循環排班 (RR) + 優先權排班 (priority)，並且針對每個不同的優先權使用單獨的佇列通常會更容易，並且 priority-scheduling 只是在最高優先權佇列中排班行程。
* 多層佇列排班法將 ready queue 區分為多個獨立的 queue。例如，一般的分類方法是區分為前台 (foreground)(交談式) 和背景 (background)(整批作業)。這兩種 process 有截然不同的回應時間 (response time)。
* foreground 和 background 不同的排班演算法。
* 五種不同佇列的多層佇列排班點算法範例，
  * 優先順序列式如下:
    1. 即時行程
    2. 系統行程
    3. 交談式行程
    4. 整批行程
  * 當即時行程、系統行程或交談式行程的佇列都已經空了，才會輪到整批行程執行。
  * 如果在整批行程的執行過程中，有交談式行程進入就緒佇列，整批行程也會讓它優先使用 CPU。
  * 當 process 進入系統之後，就會分派到某一個固定佇列，而每個佇列的 process 都不能轉移到別的佇列。
  * 優點:降低排班的負荷，缺點:沒有彈性。

### 5.3.6 多層佇列回饋排班法 (multi-level feedback queue scheduling) ###

* 相較於多層佇列排班法 (multi-level queue scheduling)，多層佇列回饋排班法允許 process 在佇列之間移動。
* 利用不同的 CPU burst 時段的特性，區分不同等級的佇列。
  * 如果一個 process 需要較長的 CPU 時間，就會排到低優先權的佇列。使得 I/O 傾向和交談式 process 放在高優先權的佇列。
  * 如果在低優先權佇列等候太久的 process，隨著時間的增長，也會漸漸地移往高優先權佇列。這種老化 (aging) 形式避免了飢餓 (starvation) 。
* 依據以下參數決定
  * 佇列個數
  * 每個佇列的盤搬演算法
  * 決定甚麼時候把 process 提升到較高優先權佇列的方法
  * 決定降低高優先權佇列的 process 到下層佇列時機的方法
  * 當 process 需要服務時，決定該 process 進入哪一個佇列的方法
* 多層佇列回饋排班法為最通用的 CPU 排班演算法，也是最複雜的一種演算法。

## 5.4 執行緒排班 (Thread Scheduling) ##

* threads 對 process 模式可分為
  * 使用者層次 (user-level) 執行緒
  * 核心層式 (kernek-level) 執行緒
* 在大多數現代 OS 上，由 OS 排程的是 kernel-level threads ，不是 process 。
* user-level thread 是由 thread library 管理，kernel 不知道 user-level thread 。
* 因此 user-level thread 為了在 CPU core 上執行，它必須映射 (mapping) 到一個相關的 kernek-level thread，雖然這個 mapping 可能是間接及可能使用輕量級行程 (lightweight process, LWP)。

### 5.4.1 競爭範圍 (Contention Scope) ###

* user-level 和 kernel-level threads 之間的差別是他們如何被排程的。
* 在製作 many-to-one 和 many-to-many 模式的系統上，thread library 排程 user-level threads 在可用的 LWP 上執行，這種技巧稱為行程競爭範圍 (process-contention scope, PCS)，因為 CPU 的競爭發生在屬於相同 process 的 thread。 (當我們說 thread library 排程 user-level threads 到可用的 LWP 上執行，並不是指 user-level threads 正在某個 CPU 上執行，這必須要 OS 排程 LWP 中的 kernel-level threads 在實體的 CPU 上執行)
* 為了決定哪一個 kernel-level threads 排程到 CPU 上，kernel 使用系統競爭範圍 (system-contention scope, SCS)。以 SCS 排班的 CPU 競爭發生在系統中的所有 threads。例如，Windows 和 Linux 等使用 one-to-one 模式的系統，只使用 SCS 排班 threads。

### 5.4.2 (Pthread 的排班) ###

## 5.5 多處理器的排班問題 (Multi-Processor Scheduling) ##

* 前面討論的重點是針對系統中 single processing core 的 CPU-Scheduling 問題。
* 如果一套系統有多個 CPU ，則可利用負載分享 (load sharing)。
* 多處理器 (multi-processor) 的系統架構
  * 傳統
    * 多個實體 processor 的系統，每個 processor 只有 single-core CPU
  * 現代
    * 多核心 CPU (Multicore CPUs)
    * 多執行緒核心 (Multithreaded cores)
    * 非統一記憶體存取架構系統 (NUMA systems)
    * 異構式多處理 (Heterogeneous multiprocessing)

### 5.5.1 多處理器排班方法 (Multiple-Processor Scheduling) ###

* 非對稱式多處理 (asymmetric multiprocessing)
  * 角色
    * 主機伺服器 (master server) : 在系統中，其中的一個 processor，擁有所有的排程決定、 I/O 處理和處理系統的其他活動。
    * 其他的 processor : 只執行使用者程式碼。
  * 只有主機伺服器中的那一個 processor 存取 system data structures ，因此減少對 data sharing 的需要。
    * 缺點 : 主機伺服器是潛在的 bottleneck ，可能造成系統效能會降低。

* 對稱式多元處理 (symmetric multiprocessing, SMP)
  * 角色
    * 每個 processor 能 self-scheduling 。經由讓每個 processor 的 scheduler 檢查 ready queue ，並選擇要運行的 threads 。
  * 兩種可能的策略來建構符合 threads 的排程
    * 一般就緒佇列 (common ready queue)
      1. 共享執行佇列 (shared run queue) 可能存在 race condition，必須確保兩個單獨的 processor 不會選擇排程同一個 thread，並且 thread 不會從 ready queue 中遺失。
      2. 可以使用鎖 (lock)，來保護 ready queue 避免 race condition。
      3. lock 將會是高度競爭的，因為對 ready queue 的所有訪問都需要 lock 的所有權，因此訪問 ready queue 可能會成為性能瓶頸。
    * 獨立核心運行柱列 (pre-core run queues)
      1. 允許 processor 從其專用執行的佇列 (private run queue) 中進行 thread 排程，因此不會面臨shared queue 造成的效能問題。
      2. 這是支援 SMP 系統最常用的方法。
      3. 每個 processor 的執行佇列 (run queue) 能更有效地使用快取記憶體 (cache memory)。
      4. 每個 processor 的 run queue 問題，每個 queue 有不同的工作負擔 (workloads)。使用平衡演算法 (balancing algorithms) 來平衡所有 processor 的 workloads 。
  * 目前幾乎所有的現代 OS 都支持 SMP，包括 Windows, Linux, macOS, Android 和 iOS。

<div style="text-align:center">
    <img src="../img/0511 - Organization of ready queues.png" alt= "0511 - Organization of ready queues.png" width="50%">
    <p>ready queue 的結構</p>
</div>

### 5.5.2 多核心處理器 (Multicore Processors) ###

* 傳統的 SMP 系統提供多個實體的 processor ；最近是將多個 CPU cores 放在同一個 chip 上，產生多核心處理器 (multicore processor)。
* multicore processor 比 multi processor 好，速度較快且消耗較少的能量。
* multicore processor 可能使 scheduling 問題複雜化。
* 記憶體停滯 (memory stall)
  * 當 processor 存取 memory，將花費時間在等待資料變成可以使用，例如:快取失誤 (存取的資料不在快取記憶體中)。
  * 這種情況下，processor 可能話費 50% 的時間等待 memory 的資料變成可以使用的。

<div style="text-align:center">
    <img src="../img/0512 - Memory stall.png" alt= "0512 - Memory stall.png" width="60%">
    <p>記憶體停滯</p>
</div>

* 處理器核心多執行緒化 (multithreaded processing cores)
  * 為了改善 memory stall ，最近許多硬體設計已經實作 multithreaded processing cores ，將兩個或多個硬體執行緒 (hardware threads) 分配到每一個 core 。
  * 如果一個 thread 因為等待記憶體而停滯，core 可以切換到另一個 thread。
  * 這種技術稱為晶片多執行緒 (chip multithreading, CMT)。
  * processor 包含 4 個 cores ，每個 core 包含兩個 thread。從 OS 的角度來看，有 8 個邏輯 CPU 。(因為 thread 是 CPU 進行 scheduling 的最小單位)

<div style="text-align:center">
    <img src="../img/0513 - Multithreaded multicore system.png" alt= "0513 - Multithreaded multicore system.png" width="60%">
    <p>多執行緒多核心系統</p>
</div>

<div style="text-align:center">
    <img src="../img/0514 - Chip multithreading.png" alt= "0514 - Chip multithreading.png" width="40%">
    <p>晶片多執行緒</p>
</div>

* Intel 處理器使用超執行緒 (hyper-threading)
  * 也稱為同步多執行緒 (simultaneous multithreading, SMT)
  * 將多個 hardware threads 分配給一個 core

* 兩種方法得到 multithreaded processing cores
  * 粗糙多執行緒 (coarse-grained multithreaded)
    * 一個 thread 會在一個 core 上執行，直到類似 memory stall 發生。
  * 精緻多執行緒 (fine-grained multithreaded)
    * 通常在指令周期的邊緣進行切換
    * 精緻系統的架構設計包含 thread switch 的邏輯，因此切換的代價較小

* core 的資源 (ex. caches, pipelines) 必須在 hardware thread 之間共享，因此 core 一次只能執行一個 hardware thread。
* 因此 multithread 和 multicore processor 需要兩種不同層級的 scheduling 。

* 兩層式排班 (two-level scheduling)
  * level 1 scheduling
    * OS 選擇一個 SW thread 在 HW thread (logical CPU) 上執行
  * level 2 scheduling
    * 設定每個 core 決定執行哪一個 HW thread

<div style="text-align:center">
    <img src="../img/0515 - Two levels of scheduling.png" alt= "0515 - Two levels of scheduling.png" width="50%">
    <p>dual-threaded processing core 的兩層式排班</p>
</div>

### 5.5.3 負載平衡 (Load Balancing) ###

* 在對稱式多元處理 (SMP) 系統中，讓所有 CPU 保持工作量平衡，才能善用 multi-processor 的優點。
* 負載平衡通常只有在每個 processor 有 private ready queue 的系統中，才是必須的。因為  common run queue 的系統，在發生 processor 發生 idle 時，將會立刻由 common ready queue 中提取一個可運行的 process。

* 兩種一般的方法
  * 推轉移 (push migration)
  * 拉轉移 (pull migration)

* 簡單的觀點
  * 只是要求所有 queue 具有大約相同數量的 threads
  * 平衡地處理需要在所有 queue 之間平均分配 threads 的 priorities

### 5.5.4 處理器親和性 (Processor Affinity) ###

* 說明
  * 避免 process 由一個 CPU core 轉移到另一個 CPU core，而是嘗試讓一個 process 在同一個 CPU core 上一直執行，這就是處理器親和性 (Processor Affinity)。

* 如果讓 process 從第一個 CPU core 搬移到第二個 CPU core 上執行，那第一個 CPU core 中 cache 的內容將變為無效，而第二個 CPU core 中 cache 必須重新填滿。

* 種類
  * 軟性親和性 (soft affinity)
  * 硬性親和性 (hard affinity)

* 非統一記憶體存取架構 (non-uniform memory access, NUMA)
  * 說明
    * 是電腦設計中，一種為 multi-core 的記憶體架構，記憶體存取時間取決於記憶體相對於處理器的位置。
    * 在 NUMA 中，CPU 存取本地記憶體比非本地記憶體的速度快。
  * 通常負載平衡 (Load Balancing) 會和處理器親和性 (Processor Affinity) 相抵銷。

<div style="text-align:center">
    <img src="../img/0516 - NUMA and CPU scheduling.png" alt= "0516 - NUMA and CPU scheduling.png" width="60%">
    <p>NUMA and CPU scheduling</p>
</div>

### 5.5.5 異構多處理 (Heterogeneous Multiprocessing, HMP) ###

* 在行動裝置系統中，CPU cores 可能會有不同的時脈速度和電源管理，包括將 core 的功率消耗調整到 idle 的程度，這種的系統稱為 HMP。
* HMP 的目的是將 task 分配給特定的 core，進行更好的功率消耗管理。

* ARM CPU 中的 big.LITTLE 架構
  * 核心種類
    * 高性能 big 核心 : 消耗能量多，只能使用較短時間
    * 節能 LITTLE 核心 : 消耗能量少，能使用較長時間
  * 優點
    * CPU scheduler 將不需要高效能但能運行較長時間的 task (ex. background task) 分配給小核心，以節省電量；將需要更多處理能力但需要較短時間的交互式 app 分配給大核心。
    * 行動裝置處於省點模式時，則可以禁止使用耗能較大的大核心，系統可以完全依靠節能性較高的小核心。

## 5.6 即時 CPU 排班 (Real-Time CPU Scheduling) ##

* 種類
  * 軟即時系統 (soft read-time system)
    * 對於非常即時的 process 沒有保證會第一個執行，只保證會比非迫切的 process 優先被考慮。
  * 硬即時系統 (haed real-time system)
    * process 必須在期限內被服務；在期限過後的服務等同於沒有被服務。

### 5.6.1 降低潛伏期 (Minimizing Latency) ###

* 說明
  * 將事件發生到它被服務時所經過的時間。

* real-time 系統是事件驅動 (event-driven) 本質，通常系統正在等待 real-time 事件的發生。
* 事件可以發生在軟體 (ex. timer) 或硬體。
* 不同的事件會有不同的潛伏期。例如:反鎖死剎車系統、飛機上控制雷達的嵌入式系統

<div style="text-align:center">
    <img src="../img/0517 - Event latency.png" alt= "0517 - Event latency.png" width="40%">
    <p>事件潛伏期</p>
</div>

* 兩種類型的潛伏期影響 real-time system 效能
  * 中斷潛伏期 (interrupt latency)
    * 指 interrupt 抵達 CPU 到開始執行中斷服務常式 (interrupt service routine, ISR) 的時間間隔
  * 分派潛伏期 (dispatch latency)
    * process 排程分派器 (dispatcher) 停止某個 process ，並啟動另一個 process 的時間量，稱為分派潛伏期

* 中斷潛伏期 (interrupt latency)
  * 當 interrupt 發生時，OS 必須先完成正在執行的 instruction，並且決定 interrupt 發生的類型，然後在使用特定的 ISR 去服務 interrupt 前，必須儲存目前 process 的狀態。

* 分派潛伏期 (dispatch latency)
  * 提供 real-time task 立即存取 CPU ，強制 real-time OS 將此潛伏期降到最低。
  * 讓分派潛伏期降低的最有效技術是提供 preemptive 的 core 。
  * 對於硬體即時系統，分派潛伏期通常是以幾微秒為單位。
  * 衝突相位 (conflict phase) 的兩個成分
    1. 任何在 core 執行的 process 為 preemptive
    2. 低優先權 process 釋出高優先權 process 需要的資源
  * 在衝突期間後，分派階段會將高優先權 process 排班到可使用的 CPU 上。

<div style="text-align:center">
    <img src="../img/0518 - Interrupt latency.png" alt= "0518 - Interrupt latency.png" width="50%">
    <p>中斷潛伏期</p>
</div>

<div style="text-align:center">
    <img src="../img/0519 - Dispatch latency.png" alt= "0519 - Dispatch latency.png" width="50%">
    <p>分派潛伏期</p>
</div>

### 5.6.2 以優先權為基礎的排班 (Priority-Based Scheduling) ###

### 5.6.3 單調速率排班法 (Rate-Monotonic Scheduling) ###

### 5.6.4 最早截止期限優先排班法 (Earliest-Deadline-First Scheduling) ###

### 5.6.5 比例分享排班法 (Proportional Share Scheduling) ###

### 5.6.6 POSIX 即時排班法 (POSIX Real-Time Scheduling) ###

## 5.7 作業系統範例 (Operating-System Examples) ##

* 在描述 Solaris 和 Windows 系統的 process scheduling 策略時，是使用 kernel threads 的排班；而在討論 Linux 排班器時，使用 task 這個名稱。

### 5.7.1 範例:Linux Scheduling ###

* 在 v2.5 前 ，Linux kernel 執行傳統 UNIX 排班演算法的變化。但是這種演算法並不是以 SMP 系統為考慮所設計，它沒有充分支援 multi-core 的系統。
* 在 v2.5 ，Linux kernel 被修改，並包含一個被稱為 O(1) 的排班演算法。
* 在 v2.6 的 kernel 開發期間，排班器再次被修改；在 v2.6.23 版發佈的 kernel，完全公平排班器 (completely fair scheduler, CFS) 變成預設的 Linux 排班演算法。

### 5.7.2 範例:Windows Scheduling ###

### 5.7.3 範例:Solaris Scheduling ###

## 5.8 演算法的評估 (Algorithm Evaluation) ##

### 5.8.1 確定性模型化 (Deterministic Modeling) ###

### 5.8.2 佇列模式 (Queueing Models) ###

### 5.8.3 模擬 (Simulations) ###

### 5.8.4 實作 (Implementation) ###

## 5.9 摘要 (Summary) ##

* CPU scheduling 是從 ready queue 選擇一個正在等待中的 process 並配置給 CPU。分派器配置許出的 process 給 CPU。
* Scheduling algorithms 可以是 preemptive 或 nonpreemptive。幾乎所有現在 OS 都是 preemptive。
* Scheduling 演算法可以根據 5 個條件作評估:(1)CPU utilization, (2)throughput, (3)turnaround time, (4)waiting time, (5)response time 。
