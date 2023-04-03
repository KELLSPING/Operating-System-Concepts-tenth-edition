# Chapter 02 -- 作業系統結構 (Operating-System Structures) #

## Chapter Objectives ##

* Identify services provided by an operating system.
  * 認識作業系統提供的服務
* Illustrate how system calls are used to provide operating system services.
  * 說明系統呼叫如何提供作業系統服務
* Compare and contrast monolithic, layered, microkernel, modular, and
hybrid strategies for designing operating systems.
  * 比較和對比用於設計作業系統的單片、分層、微核心、模組和混合策略
* Illustrate the process for booting an operating system.
  * 說明啟動作業系統的過程
* Apply tools for monitoring operating system performance.
  * 應用工具來監視作業系統性能
* Design and implement kernel modules for interacting with a Linux kernel.
  * 設計和實現用於與 Linux 核心交互的核心模組

## Section ##

* [2.1 作業系統服務 (Operating-System Services)](#21-作業系統服務-operating-system-services)
* [2.2 使用者與作業系統介面 (User and Operating-System Interface)](#22-使用者與作業系統介面-user-and-operating-system-interface)
* [2.3 系統呼叫 (System Calls)](#23-系統呼叫-system-calls)
* [2.4 系統服務 (System Services)](#24-系統服務-system-services)
* [2.5 鏈結器與載入器 (Linkers and Loaders)](#25-鏈結器與載入器-linkers-and-loaders)
* [2.6 為什麼應用程式是特定作業系統 (Why Applications Are Operating-System Specific)](#26-為什麼應用程式是特定作業系統-why-applications-are-operating-system-specific)
* [2.7 作業系統的設計和製作 (Operating-System Design and Implementation)](#27-作業系統的設計和製作-operating-system-design-and-implementation)
* [2.8 作業系統結構 (Operating-System Structure)](#28-作業系統結構-operating-system-structure)
* [2.9 構建和啟動作業系統 (Building and Booting an Operating System)](#29-構建和啟動作業系統-building-and-booting-an-operating-system)
* [2.10 作業系統除錯 (Operating-System Debugging)](#210-作業系統除錯-operating-system-debugging)
* [2.11 摘要 (Summary)](#211-摘要-summary)

## 2.1 作業系統服務 (Operating-System Services) ##

<div style="text-align:center">
    <img src="img/02_01-operating_system_services.png" alt= “02_01-operating_system_services” width="70%">
</div>

* 使用者介面 (User Interface, UI)
  * 圖形使用者介面 (Graphical User Interface, GUI)
  * 觸控螢幕介面 (Touch-Screes Interface)
  * 命令行介面 (Command-Line Interface, CLI)

* 系統呼叫 (System Calls)

* 作業系統服務 (Operating-System Services)
  * 對使用者有幫助的功能
    * 程式執行 (Program execution)
    * I/O 作業 (I/O operations)
    * 檔案系統的使用 (File-system manipulation)
    * 通信 (Communications)
      1. 共用記憶體 (Shared Memory)
      2. 訊息傳遞 (Message Passing)
    * 錯誤偵測 (Error detection)
  * 為了確保系統能有效運作
    * 資源分配 (Resource allocation)
    * 紀錄檔記錄 (Logging)
    * 保護和安全 (Protection and security)

## 2.2 使用者與作業系統介面 (User and Operating-System Interface) ##

* 命令行列介面 (CLI), 命令解譯器 (Command Interpreter)
  * 又可稱為外殼 (Shell)
    1. C shell
    2. Bourne-Again (或 bash) shell
    3. Korn shell
  * 在 UNIX 中，命令解譯器並不了解命令的涵義，它是使用命令來指定一個要載入記憶體及執行的檔案。
    * rm file.txt => 尋找叫作 rm 的檔案，將它載入記憶體，並將參數 file.txt 傳給他。
* 圖形使用者介面 (Graphical User Interface, GUI)
  * 使用圖像 (Icon) 代表程式、檔案、目錄和系統功能。
  * 第一個 GUI 是 1973 年出現在 Xerox Alto 電腦上。
  * 在 UNIX 或 Linux 系統上執行的 K Desktop Environment (KDE) 或 GNOME 桌面。
* 觸控螢幕介面 (Touch-Screes Interface)
  * 利用手勢 (Gesture) 來進行互動。
* 選擇介面
  * 管理電腦的系統管理員 (System Administrator) 和對系統比較了解的超級使用者 (Power User) 通常會使用 CLI 。
  * 外殼劇本 (Shell Scripts)
    * 一組命令步驟，記錄在檔案中，不會被編譯成執行碼，而是由 CLI 直接解譯。

## 2.3 系統呼叫 (System Calls) ##

* 提供一個由 Operating-System Services 的介面。
* 一般以 C 或 C++ 寫成的函數。如果系統呼叫是比較低階的工作 (ex.硬體須直接存取) 將可能要以組合語言指令來寫。
* 應用程式設計界面
  * 應用程式開發人員依照應用程式介面 (Application Programming Interface, API) 設計程式。
  * 應用程式設計者三個最常用的 API
    1. Windows API
    2. 以 POSIX 系統為基礎的 POSIX API (包括所有的 UNIX, Linux, macOS 的版本)
    3. Java 虛擬機上執行 Java 程式的 Java API
  * 在 UNIX 和 Linux 等程式是使用 C 語言寫的情況下，此函數庫稱為 libc 。
  * 執行時間環境 (Run-time environment, RTE)
  * 系統呼叫介面 (System-call Interface)
* 系統呼叫的類型可分為 6 大類
  1. 行程控制 (Process Control)
  2. 檔案管理 (File Management)
  3. 裝置管理 (Device Management)
  4. 資訊維護 (Information Maintenance)
  5. 通信 (Communication)
  6. 保護 (Protection)
  
## 2.4 系統服務 (System Services) ##

* 又稱為系統常式 (System Utilities)
* 提供程式開發與執行的便利環境
* 分類
  1. 檔案管理
  2. 狀態資訊
  3. 檔案修改
  4. 程式語言支援
  5. 程式的載入與執行
  6. 通信
  7. 背景服務
      * 一直執行的系統程式行程被稱為服務 (Services)、子系統 (Subsystems) 或守護程序。

## 2.5 鏈結器與載入器 (Linkers and Loaders) ##

<div style="text-align:center">
    <img src="img/02_11-linker_and_loader.png" alt= “02_11-linker_and_loader” width="45%">
</div>

* 可重定位物件檔案 (Relocatable Object File, .o file)
  * 來源檔案 (Source file, .c file) 被編譯器 (Compiler) 編譯成物件檔案，這些物件檔案被設計成可以載入到任何實體記憶體的位置，這種格式稱為可重定位物件檔案。
* 可執行的檔案 (Executable File, .exe file)
  * 鏈結器 (Linker) 將這些可重定位的物件檔案組合為單個二進制可執行的檔案。
* 載入器 (Loader)
  * 用於將二進制可執行檔案載入到記憶體中，該檔案可以在 CPU 核心上執行。
  * 要執行 Loader ，在命令列中輸入可執行檔案的名稱即可。
  * 與 Linker 和 Loader 相關的程序是重定位 (Relocation)。

## 2.6 為什麼應用程式是特定作業系統 (Why Applications Are Operating-System Specific) ##

## 2.7 作業系統的設計和製作 (Operating-System Design and Implementation) ##

## 2.8 作業系統結構 (Operating-System Structure) ##

## 2.9 構建和啟動作業系統 (Building and Booting an Operating System) ##

## 2.10 作業系統除錯 (Operating-System Debugging) ##

## 2.11 摘要 (Summary) ##
