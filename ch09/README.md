# Part 04 - 記憶體管理 (Memory Management) #

* 電腦系統的主要目的是執行 programs 。
* programs 與其存取的資料在執行期間，至少有部分必須在 main memory 中。
* 現代電腦系統在執行期間必須保持多個 processes 在 main memory 中。
* Memory management 的方法有很多種，這些方法反映各種不同的途徑及有效性。
* 為特定之系統選擇某一 memory management 方法依很多情況而異，特別是依系統的硬體設計而定。
* 大部分 memory management 方法的演算法需要有某些形式的硬體支援。

## Chapter 09 -- 主記憶體 (Main Memory) ##

* 

## Section Goals ##

* Explain the difference between a logical and a physical address and the role of the memory management unit (MMU) in translating addresses.
  * 探討邏輯位址和實體位址的差異，以及記憶體管理單元(MMU)在轉換位址中的角色
* Apply first-, best-, and worst-fit strategies for allocating memory contiguously.
  * 探討最先(first)配合、最佳(best)、最差(worst)配合策略來連續配置記憶體
* Explain the distinction between internal and external fragmentation.
  * 說明內部斷裂與外部斷裂的差異
* Translate logical to physical addresses in a paging system that includes a translation look-aside buffer (TLB).
  * 透過分頁系統包括轉譯旁觀緩衝區(TLB)將邏輯位址轉換到實體位址
* Describe hierarchical paging, hashed paging, and inverted page tables.
  * 描述階層式分頁、雜湊分頁和反轉分頁表
* Describe address translation for IA-32, x86-64, and ARMv8 architectures.
  * 描述 IA-32、X86-64 和 ARMv8 架構的位址轉換

## Section ##

* [9.1 背景說明 (Background)](#91-背景說明-background)
* [9.2 連續記憶體配置 (Contiguous Memory Allocation)](#92-連續記憶體配置-contiguous-memory-allocation)
* [9.3 分頁 (Paging)](#93-分頁-paging)
* [9.4 分頁表的結構 (Structure of the Page Table)](#94-分頁表的結構-structure-of-the-page-table)
* [9.5 置換 (Swapping)](#95-置換-swapping)
* [9.6 範例: Intel 32 和 64 位元架構 (Example: Intel 32\-bit and 64\-bit Architectures)](#96-範例-intel-32-和-64-位元架構-example-intel-32-bit-and-64-bit-architectures)
* [9.7 範例: ARM 架構 (Example: ARMv8 Architecture)](#97-範例-arm-架構-example-armv8-architecture)
* [9.8 摘要 (Summary)](#98-摘要-summary)

## 9.1 背景說明 (Background) ##

## 9.2 連續記憶體配置 (Contiguous Memory Allocation) ##

## 9.3 分頁 (Paging) ##

## 9.4 分頁表的結構 (Structure of the Page Table) ##

## 9.5 置換 (Swapping) ##

## 9.6 範例: Intel 32 和 64 位元架構 (Example: Intel 32\-bit and 64\-bit Architectures) ##

## 9.7 範例: ARM 架構 (Example: ARMv8 Architecture) ##

## 9.8 摘要 (Summary) ##
