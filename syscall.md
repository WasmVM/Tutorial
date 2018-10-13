# 系統呼叫

系統呼叫不在 WebAssembly 規格內，是 WasmVM 特有的功能，可以讓 WebAssembly 直接對作業系統輸出/輸入等等的操作

系統呼叫的機制是先在堆疊中放入指定的參數，再透過 unreachable 指令調用系統呼叫

系統呼叫目前只支援 Linux 作業系統，Windows 和 Mac 無法使用

## 編譯 WasmVM

先前在編譯時有把系統呼叫功能關掉，現在要把他打開

* 到 build 資料夾 \(如果已經在 build 資料夾可以跳過這一步\)

  ```text
  cd build
  ```

* 執行 CMake

```bash
cmake -DCMAKE_BUILD_TYPE=Debug -DUSE_SYSCALL=ON ..
```

一樣使用 Debug 模式，但是把系統呼叫打開

**在打開系統呼叫之後，unreachable 就會用來調用系統呼叫，不能再拿來輸出堆疊中的數值**

* 執行 Make 編譯程式

```text
make -B
```

## 使用系統呼叫

在這個簡化版中，只有 3 個系統呼叫

一般版中會支援更多系統呼叫，使用方式基本上相差不大

### SYS\_stdin

從終端機的標準輸入，讀取資料到記憶體

* 編號: 0
* 參數

| 名稱 | 型別 |
| :--- | :--- |
| 記憶體位置 | i32 |
| 讀取長度 | i32 |

* 回傳值
  * 一個 i32 數值，成功的話回傳實際讀取長度，失敗的話回傳 -1

範例程式

```text
(module
    (memory 1)
    (func $main (local i32)
          i32.const 0
          i32.const 10
        i32.const 0
        unreachable
        drop
        i32.const 0
          i32.const 10
        i32.const 1
        unreachable
        drop
    )
    (start $main)
)
```

第一個 i32.const 是記憶體位置 0

第二個 i32.const 讀取長度 10

第三個 i32.const 是 SYS\_stdin 的編號 0

執行後會把鍵盤輸入的字顯示出來

### SYS\_stdout

將記憶體中的資料，寫到終端機的標準輸出

* 編號: 1
* 參數

| 名稱 | 型別 |
| :--- | :--- |
| 記憶體位置 | i32 |
| 讀取長度 | i32 |

* 回傳值
  * 一個 i32 數值，成功的話回傳實際寫入長度，失敗的話回傳 -1

範例程式

```text
(module
    (memory (data "Hello\n"))
    (func $main
        i32.const 0
          i32.const 6
        i32.const 1
        unreachable
        drop
    )
    (start $main)
)
```

第一個 i32.const 是記憶體位置 0

第二個 i32.const 讀取長度 6

第三個 i32.const 是 SYS\_stdout 的編號 1

執行結果

```text
Hello
```

### SYS\_stderr

將記憶體中的資料，寫到終端機的標準錯誤輸出

* 編號: 2
* 參數

| 名稱 | 型別 |
| :--- | :--- |
| 記憶體位置 | i32 |
| 讀取長度 | i32 |

* 回傳值
  * 一個 i32 數值，成功的話回傳實際寫入長度，失敗的話回傳 -1

範例程式

```text
(module
    (memory (data "Hello\n"))
    (func $main
        i32.const 0
          i32.const 6
        i32.const 2
        unreachable
        drop
    )
    (start $main)
)
```

第一個 i32.const 是記憶體位置 0

第二個 i32.const 讀取長度 6

第三個 i32.const 是 SYS\_stderr 的編號 2

執行結果

```text
Hello
```

