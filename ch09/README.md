# Part 04 - 記憶體管理 (Memory Management) #

* 電腦系統的主要目的是執行 programs 。
* programs 與其存取的資料在執行期間，至少有部分必須在 main memory 中。
* 現代電腦系統在執行期間必須保持多個 processes 在 main memory 中。
* Memory management 的方法有很多種，這些方法反映各種不同的途徑及有效性。
* 為特定之系統選擇某一 memory management 方法依很多情況而異，特別是依系統的硬體設計而定。
* 大部分 memory management 方法的演算法需要有某些形式的硬體支援。

## Chapter 09 -- 主記憶體 (Main Memory) ##

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

* memory 本身是一個大型的 bytes array ，每個 bytes 都有自己的 address 。
* CPU 是根據 program counter 的數值去 memory 的位址擷取 instructions ，這些 instructions 可能會造成特殊 memory addresses 額外的載入或儲存的動作。
* 一個典型的 instruction-execution 週期
  1. 從 memory 中取出一個 instruction
  2. 解碼 instruction ，並且可能造成 operand 從 memory 中被取出
  3. 執行完在 operand 上面的 instruction 後，可能會將結果存回 memory 中。
* memory unit 看到的只是序列的 memory addresses ， memory unit 並不知道 instruction (ex. 指令計數器、索引、間接、實字位址) 代表什麼意思，也不知道 instruction 或 data 是為甚麼產生的。
* 因此可以忽略 programs 是如何產生 memory addresses 的，只要關心執行中 programs 產生 memory addresses 的順序。
* 管理 memory 的各種不同相關技巧
  * 基本硬體、符號 (symbolic) 或虛擬 (virtual) memory addresses 與 physical addresses 的連結
  * 邏輯位址 (logical address) 和實體位址 (physical addresses) 間區分
  * 動態載入鏈結程式碼 (dynamic linking) 和共用程式庫 (shared libraries)

### 9.1.1 基本硬體 (Basic Hardware) ###

* CPU 只能直接存取 (direct access) main memory 和 processing core 中的 registers 這兩種 general-purpose storage 。
* 機器指令使用 memory addresses 作為參數，但沒有包括使用 disk addresses 。
* 任何執行的 instructions 和任何被這些 instructions 使用的 data 必須放在 main memory 或 core 的 registers 中；如果 data 不在 memory 中，則必須在 CPU 操作 data 之前，先將它們移到 memory 中。
* memory 的存取速度相較於 register 是非常慢的，造成 processor 在存取時需要停頓 (stall) ，因此補救速度上差異的記憶體緩衝器稱為快取記憶體 (cache) 。
* 在 memory 停頓期間，multithreaded core 可以從停頓的 hardware thread 切換到另一個 HW thread 。
* 對於正常的系統操作，必須保護 OS 免於 user processes 的存取；也保護 user processes 之間不互相影響。這種保護必須由硬體提供。
* 每一個 process 都有個別 memory space 以保護彼此，且讓多個 process 載入 memory 以 concurrent 執行是基本的。
* 讓各個 process 的 memory space 分開，需要有決定 process 存取合法 address 範圍的能力。經由使用兩個暫存器來提供保護
  * 基底暫存器 (base register) : 存放最小合法的實體記憶體位址
  * 界線暫存器 (limit register) : 含有範圍的大小
* base register 和 limit register 可由 OS 使用一個特殊的特權 instruction 來設定。特權 instruction 只能在 kernel mode 中執行，而且只有 OS 能在 kernel mode 之中執行，因此只有 OS 能設定 base register 和 limit register 的值，防止 user program 進行改變。
* 在 kernel mode 執行的 OS ，可以對 OS memory 和 user memory 進行存取。
  * 讓 OS 可以將 user program 載入 user memory，如果發生錯誤，則可以將 user program 傾印出來。
  * 存取和更改系統呼叫的參數。
  * 對 user memory 執行 I/O

<div style="text-align:center">
    <img src="../img/0901 - A base and a limit register define a logical address space.png" alt= "0901 - A base and a limit register define a logical address space.png" width="40%">
    <p>基底暫存器和界線暫存器定義一個邏輯位址空間</p>
</div>

<div style="text-align:center">
    <img src="../img/0902 - Hardware address protection with base and limit registers.png" alt= "0902 - Hardware address protection with base and limit registers.png" width="60%">
    <p>使用基底和界線暫存器的硬體位址保護</p>
</div>

## 9.2 連續記憶體配置 (Contiguous Memory Allocation) ##

## 9.3 分頁 (Paging) ##

## 9.4 分頁表的結構 (Structure of the Page Table) ##

## 9.5 置換 (Swapping) ##

## 9.6 範例: Intel 32 和 64 位元架構 (Example: Intel 32\-bit and 64\-bit Architectures) ##

## 9.7 範例: ARM 架構 (Example: ARMv8 Architecture) ##

## 9.8 摘要 (Summary) ##
