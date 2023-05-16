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
  1. 互斥 (mutex exclusion) : 當有 process 在 critical section 內執行，則其他的 process 不能在其 critical section 內執行。
  2. 進行 (progress) : 當一個 process 想要進入 critical section ， 除了 critical section 內沒有 process 在執行， 這個 process 也不能在 remainder section ， 才能在下一次進入 critical section ，並且這個選擇不得無限期地延遲。
  3. 限制性的等待 (bounded waiting) : 在一個 process 已經要求進入其 critical section ， 而此要求尚未被答應之前，允許其他 process 進入其 critical section 的次數有一個限制。

```C
while (true) {
    [入口區段]
        critical section
    [出口區段]
        remainder section
}
```

* 在指定時間點上，許多 kernel mode 的 process 可能在 OS 中動作，因此實現 OS 的核心程式碼會有一些可能的 race condition 。
  * 例如:分配 PID 的 race condition

<div style="text-align:center">
    <img src="../img/0602 - Race condition when assigning a pid.png" alt= "0602 - Race condition when assigning a pid.png" width="60%">
    <p>當分配 PID 的競爭條件</p>
</div>

* 在 single core 環境中，只要可以防止在修改共享變數時發生中斷，便可以簡單地解決 critical section 問題。
* 在 multi core 環境中，禁止使用中斷可能會很耗時，因為消息將傳遞到所有的 core 中，此消息延遲傳入各個 critical section ，造成系統效率下降。
* 如果系統時鐘是靠中斷更新的，那麼也要考慮到禁用中斷對系統時鐘造成的影響。

* 2 種方法用來處理 OS 中 critical section 的方法
  * 可搶先核心 (preemptive kernel)
    * 讓一個 process 在 kernel mode 中執行時可以被搶先。
    * 必須小心設計，以確保共用的核心資料沒有 race condition 。
  * 不可搶先核心 (nonpreemptive kernel)
    * 不允許一個 process 在 kernel mode 執行時被搶先。
    * 每次只有一個 process 在 kernel 中有動作，可免於核心資料結構上的 race condition 。

## 6.3 Peterson 解決問題 (Peterson’s Solution) ##

## 6.4 同步的硬體支援 (Hardware Support for Synchronization) ##

* 以軟體為基礎的的方式解決 critical section 的問題
  * 稱為基於軟體的解決方案，因為該演算法涉及 OS 的任何特殊支援或特定的硬體說明，以確保互斥。
  * 無法保證在現代的電腦能運作
* 3 個硬體指令來解決 critical section 問題
  1. 記憶體屏障 (Memory Barriers)
  2. 硬體指令 (Hardware Instructions)
  3. 單元變數 (Atomic Variables)

### 6.4.1 記憶體屏障 (Memory Barriers) ###

* 記憶體模型 (memory model)
  * 電腦架構如何確定它將向應用程式提供記憶體保證

* 記憶體模型屬於以下 2 類之 1
  * 強排序 : 在一個 processor 上進行 memory 修改時，其他 processor 立即可知道
  * 弱排序 : 在一個 processor 上進行 memory 修改時，其他 processor 不會立即知道

### 6.4.2 硬體指令 (Hardware Instructions) ###

### 6.4.3 單元變數 (Atomic Variables) ###

## 6.5 互斥鎖 (Mutex Locks) ##

* 6.4 節，對 critical section 問題以硬體為基礎的解決很複雜，並且是應用程式設計師無法接觸到的。
* OS 設計者建立軟體工具以解決 critical section 問題。軟體工具中，最簡單的是互斥鎖 (mutex lock)。
* 互斥鎖 (mutual exclusion lock, mutex lock)
  * 使用 mutex 來保護 critical section ，避免 race condition 。
  * process 在進入 critical section 前，必須取得 lock ；當 process 離開 critical section 前，會釋放 lock 。
  * 函數 acquire() 取得鎖；函數 release() 釋放鎖。

  ```C
  /* 使用 mutex 解決 critical section 問題 */
  while (true) {
      [acquire lock]
          critical section
      [release lock]
          remainder section
  }
  ```

* 鎖的競爭 (lock contention)
  * lock 可以是競爭的 (contended) ，也可以是無競爭的 (uncontended) 。
    * 競爭的鎖 : 如果嘗試取得 lock 時， thread 會被阻塞，該 lock 被認為是 contended 。
      1. 高競爭 (high contention) : 嘗試獲取 lock 時， thread 的數量較多
      2. 低競爭 (low contention) : 嘗試獲取 lock 時， thread 的數量較少
    * 無競爭的鎖 : 如果嘗試取得 lock 時， thread 不會被阻塞，該 lock 被認為是 uncontended 。
  * 高競爭的鎖 (highly contended locks) 會降低 concurrent 應用程式的整體效能。

* 時序時間 (short duration)
  * 在 multiprocessor 系統中，持續時間內保持鎖定的機制會被認為是 Spinlocks 。
  * 持續時間 : 需要等待 2 個內容轉換器時間的鎖
    * 其中 1 個內容轉換器 : 用於將 thread 移至 waiting state
    * 另 1 個內容轉換器 : 一旦 lock 釋放後，用於恢復 waiting thread

### 自旋鎖 (Spinlocks) ###

* mutex 通常使用 6.4 節的 compare_and_swap() (CAS) 操作來實現。 它要求忙碌等待 (busy waiting) ，當一個 process 在它的 critical section 時，任何試圖進入 critical section 的其他 process 必須在呼叫 acquire() 的地方不斷地執行。
* 在真正的 multiprogramming 系統中，多個的 process 共享一個 CPU core ，busy waiting 的這種連續循環顯然是一個問題，並且還會浪費 CPU cycles。
* 6.6 節，使用一種方法，透過暫時讓 waiting process 進入 sleep，然後在 lock 可用時，將其喚醒，來避免 busy waiting 。
* 在 mutex 中，使用 CAS 操作來時間，這種型態的 mutex 也被稱為 spinlock ， 因為 process 在等待 lock 可以取得時，一直"盤旋"著。
* 優點
  * 當 process 必須等待一個 lock 時，不需要 content switch 。 (content switch 很費時)
  * 在 multicore 系統中，spinlock 是很有用的，當一個 lock 在持續時間被把持時，一個 thread 可以 spin 在一個 CPU core 上，而另一個 thread 則在其他 CPU core 執行 critical section。

## 6.6 號誌 (Semaphores) ##

## 6.7 監控器 (Monitors) ##

## 6.8 存活 (Liveness) ##

## 6.9 評估 (Evaluation) ##

## 6.10 摘要 (Summary) ##
