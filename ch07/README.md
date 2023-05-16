# Chapter 07 -- 同步範例 (Synchronization Examples) #

## 在 ch6 中 ##

* 介紹了 critical-section 問題，並著重討論當多個 concurrent processes 共享資料時出現的 race condition 。
* "預防 race conditions 來解決 critical-section 問題"的工具
  * 低階硬體解決方案
    * memory barriers
    * compare-and-swap operation (CAS)
  * 高階的工具
    * mutex locks
    * semaphores
    * monitors
* 設計沒有 race conditions 的應用程序的各種挑戰，包括 liveness hazards 如 deadlocks 等。

## 在 ch7 中 ##

* 將 ch6 介紹的工具應用於經典的 synchronization 問題。
* 探討 Linux, UNIX, Windows 使用的 synchronization 機制，並描述 Java 和 POSIX 系統中 API 的詳細資訊。

## Section Goals ##

* Explain the bounded-buffer, readers –writers, and dining –philosophers synchronization problems.
  * 解釋有限緩衝區、讀寫器和哲學家進餐的同步問題
* Describe specific tools used by Linux and Windows to solve process synchronization problems.
  * 描述 Linux 和 Windows 用於解決行程同步問題的特定工具
* Illustrate how POSIX and Java can be used to solve process synchronization problems.
  * 說明如何使用 POSIX 和 Java 解決行程同步問題
* Design and develop solutions to process synchronization problems using POSIX and Java APIs.
  * 設計和開發解決方案，以使用 POSIX 和 Java API 處理行程同步問題

## Section ##

* [7.1 典型的同步問題 (Classic Problems of Synchronization)](#71-典型的同步問題-classic-problems-of-synchronization)
* [7.2 核心的同步 (Synchronization within the Kernel)](#72-核心的同步-synchronization-within-the-kernel)
* [7.3 POSIX 的同步 (POSIX Synchronization)](#73-posix-的同步-posix-synchronization)
* [7.4 Java 同步 (Synchronization in Java)](#74-java-同步-synchronization-in-java)
* [7.5 替代方法 (Alternative Approaches)](#75-替代方法-alternative-approaches)
* [7.6 摘要 (Summary)](#76-摘要-summary)

## 7.1 典型的同步問題 (Classic Problems of Synchronization) ##

## 7.2 核心的同步 (Synchronization within the Kernel) ##

## 7.3 POSIX 的同步 (POSIX Synchronization) ##

## 7.4 Java 同步 (Synchronization in Java) ##

## 7.5 替代方法 (Alternative Approaches) ##

## 7.6 摘要 (Summary) ##
