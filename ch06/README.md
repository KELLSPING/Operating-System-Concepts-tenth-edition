# Part 03 - 行程同步 (Process Synchronization) #

* 一個系統包含多個或大量的 threads ，其中包含並行 (concurrently) 或是平行 (parallel) 。
* threads 通常共享資訊，同時 OS 不斷更新不同的 data structures 來支援 multi-threads。
* 競爭情況 (race condition) 會在存取共享資料無法被控制時出現。
* 行程同步 (process synchronization) 使用了工具來控制存取共享資料，避免 race condition。工具必須小心使用，以免導致系統效能下降，包括死結 (dead lock)。

## Chapter 06 -- 同步工具 (Synchronization Tools) ##

* 合作行程 (cooperating process)
  * cooperating processes 可以影響或被影響系統中其他執行的 process。
  * cooperating processes 之間可能直接分享一塊邏輯位址空間 (ex. code, data) ，或是只能經由檔案分享資料 (ex. shared memory, message passing)。
  * 同時 (concurrent) 存取共用資料可能造成資料前後不一致。

## Section Goals ##

* Describe the critical-section problem and illustrate a race condition.
  * 描述臨界區間問題，並說明競爭狀況
* Illustrate hardware solutions to the critical-section problem using memory barriers, compare-and-swap operations, and atomic variables.
  * 透過記憶體屏障、比較和交換操作和原子變數來進行關於臨界區間問題的硬體答案
* Demonstrate how mutex locks, semaphores, monitors, and condition variables can be used to solve the critical-section problem.
  * 使用互斥鎖、號誌、監控器和條件變數解決臨界區間問題
* Evaluate tools that solve the critical-section problem in low-, moderate-, and high-contention scenarios.
  * 評估解決低、中度和高競爭的方案

## Section ##

* [6.1 背景 (Background)](#61-背景-background)
* [6.2 臨界區間問題 (The Critical-Section Problem)](#62-臨界區間問題-the-critical-section-problem)
* [6.3 Peterson 解決問題 (Peterson’s Solution)](#63-peterson-解決問題-petersons-solution)
* [6.4 同步的硬體支援 (Hardware Support for Synchronization)](#64-同步的硬體支援-hardware-support-for-synchronization)
* [6.5 互斥鎖 (Mutex Locks)](#65-互斥鎖-mutex-locks)
* [6.6 號誌 (Semaphores)](#66-號誌-semaphores)
* [6.7 監控器 (Monitors)](#67-監控器-monitors)
* [6.8 存活 (Liveness)](#68-存活-liveness)
* [6.9 評估 (Evaluation)](#69-評估-evaluation)
* [6.10 摘要 (Summary)](#610-摘要-summary)

## 6.1 背景 (Background) ##

* 3.2.2 節，介紹 process scheduling 的角色，並描述 CPU-scheduler 在 processor 間快速切換以提供 concurrent 執行 ，這表示一個 process 在另一個 process 被排班前可能只有部分地完成執行。
* 4.2 節，介紹 parallel 執行， 兩個不同的 process 同時在各自的 CPU core 中執行。
* 6.1 節，介紹 concurrent 和 parallel 執行如何貢獻關於多行程共用資料的完整性。
* 描述有限緩衝問題 (Bounded-buffer problem) 如何讓 process 共用記憶體。
* 第 6 章的主要部分是在專注於 cooperating process 間的行程同步 (process Synchronization) 和協調 (coordination) 問題。

## 6.2 臨界區間問題 (The Critical-Section Problem) ##

* 說明
  * 在同步的程式設計中， critical section 指的是 process 中，一個存取共享資源的程式片段，而這些共享資源無法同時被多個 thread 存取的特性。
* critical section Problem 就是設計一套協定，使得各個 process 能夠互相合作，每個 process 必須得到允許，才能進入臨界區間。
* 名詞
  * 入口區段 (entry section)
  * 出口區段 (exit section)
  * 臨界區段 (critical section, CS)
  * 剩餘區段 (remainder section, RS)
* 3 項要求解決臨界區兼問題
  * 互斥 (mutex exclusion) : 當有 process 在 critical section 內執行，則其他的 process 不能在其 critical section 內執行。
  * 進行 (progress) : 當一個 process 想要進入 critical section ， 除了 critical section 內沒有 process 在執行， 這個 process 也不能在 remainder section ， 才能在下一次進入 critical section ，並且這個選擇不得無限期地延遲。
  * 限制性的等待 (bounded waiting) : 在一個 process 已經要求進入其 critical section ， 而此要求尚未被答應之前，允許其他 process 進入其 critical section 的次數有一個限制。

## 6.3 Peterson 解決問題 (Peterson’s Solution) ##

## 6.4 同步的硬體支援 (Hardware Support for Synchronization) ##

## 6.5 互斥鎖 (Mutex Locks) ##

## 6.6 號誌 (Semaphores) ##

## 6.7 監控器 (Monitors) ##

## 6.8 存活 (Liveness) ##

## 6.9 評估 (Evaluation) ##

## 6.10 摘要 (Summary) ##
