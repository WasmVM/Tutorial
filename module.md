# 模組

終於進入到重頭戲了！前面我們常常提到模組，模組是 WebAssembly 程式發佈、載入和執行的基本單位。依據執行時期的不同，分別有2種不同的結構

## 一般架構

模組在打包成位元檔，以及剛載入時的架構

在 WasmVM 裡，一個檔案只能包含一個模組

一般架構的模組分成以下幾個部份：

### 函式型別 \(Types\)

這邊的函式型別和之前講的 i32、i64.....等等的數值型別不一樣，是表示函式的參數\(Parameter\)和回傳值種類。在一些程式語言中會被稱為函式簽名 \(Function Signature\)

```text
(module
    (type (func (param i32) (result i32)))
    (type $void (func))
    (func $a1 (type 0)
        get_local 0
    )
    (func $a2 (param i32) (result i32)
        get_local 0
    )
    (func $a3 (result i32)
        i32.const 5
    )
    (func $main (type $void)
          i32.const 3
        call $a1
          i32.const 4
        call $a2
        call $a3
        unreachable
    )
    (start $main)
)
```

我們宣告了 2 種函數型別，並且讓下面的 $a1 函數使用第 1 個\(0 號\)型別。

$a2 雖然沒有用我們宣告的型別，不過因為回傳值的型別，以及參數的型別和種類都與 0 號型別一樣，所以會自動套用 0 號型別

$a3 的函數型別沒有宣告，這個型別會接著有宣告的編號，自動被宣告成 2 號型別

和 memory 或 table 一樣，函式行別也能指定名稱。$main 函式就用了 $void 這個型別

### 函式 \(Functions\)

這部份在 [函式](store/function.md) 章節已經有詳細說明

### 函式表 \(Tables\)

這部份在 [函式表](store/table.md) 章節已經有詳細說明

### 記憶體 \(Memories\)

這部份在 [記憶體](store/memory.md) 章節已經有詳細說明

### 全域變數 \(Global Variables\)

這部份在 [變數](store/variables.md) 章節已經有詳細說明

### 函式表元素 \(Elements\)

這部份在 [函式表](store/table.md) 章節已經有詳細說明

### 記憶體資料 \(Data\)

可以利用這個部份，填入記憶體最初始的資料

```text
(module
    (memory 0)
    (data 0 (i32.const 4) "hahaha")
)
```

第一個 0 表示第 0 號記憶體，因為在目前的規格中一個模組只能有一塊記憶體，所以這邊只能填 0 ，或是記憶體的名稱

後面的 i32.const 表示要從記憶體的哪個位置開始填，這邊只能用 i32.const，不能用其他指令

接著是初始資料的字串，在 [記憶體](store/memory.md) 章節有詳細的說明

當記憶體的長度不夠時，如果沒超出記憶體最大值，或是記憶體沒有最大值，會自動增加到足夠的長度

### 起始函式 \(Start function\)

表示當這個模組作為程式的主要模組時，首先呼叫的函式

這個函式不能有參數，也不能有回傳值

在之前的範例中是指定函數名稱 $main，其實也不一定要叫作 $main，可以用各種名稱，甚至只用編號也行

```text
(module
    (func
    )
    (start 0)
)
```

### 引入 \(Imports\)

這個部份可以從其他檔案載入模組，並從其他模組中引入函式、函式表、記憶體或全域變數

```text
(module
    (import "test2.wasm" "fun" (func))
    (func $main
        call 0
        i32.const 1
        unreachable
    )
    (start $main)
)
```

第一個字串是要引入的模組名稱\(在 WasmVM 裡就是檔案名稱，去掉資料夾路徑\)，如果這個檔案沒被載入過，WasmVM 會自動從目前的工作資料夾中尋找這個檔案，並載入其中的模組。已經載入過的模組會直接從載入的資料中提取要引入的部份，不會重複載入。

第二個字串是物件的輸出名稱。在輸出時會指定一個輸出名稱給要輸出的物件，這邊必須要和輸出時指定的輸出名稱相符

後面則是要引入的物件型別:

* 函式：照函式型別的格式，如果有的話，必須包含 param 和 result，不需要 local 和 裏面的指令
* 函式表：照 table 的宣告格式，必須包含最小值和 anyfunc，如果有指定的話要包含最大值
* 記憶體：照 memory 的宣告格式，必須包含最小值，如果有指定的話要包含最大值，不需要 data
* 全域變數： \(global 數值型別\)

以上物件都可以指定以 $ 開頭的名稱，加在 table, memory, func 或 global 後面

上面範例中引入的 test2.wasm

```text
(module
    (func
        i32.const 2
        unreachable
    )
    (export "fun" (func 0))
)
```

輸出結果

```text
Values in the stack:
Type: i32, Value: 2
Values in the stack:
Type: i32, Value: 1
```

#### 注意事項

1. 一個模組最多只能有一個函式表、一個記憶體，所以如果已經引入函式表、記憶體，就不能再引入或宣告函式表或記憶體讓數量超過；如果已經宣告函式表、記憶體，也不能再引入或宣告函式表或記憶體讓數量超過
2. 目前的規格中只能引入不可變動的全域變數

### 輸出 \(Exports\)

這個部份可以輸出模組中的函式、函式表、記憶體或全域變數，讓其他模組引入

```text
(module
    (func
        i32.const 2
        unreachable
    )
    (export "fun" (func 0))
)
```

語法上和引入的差別在於 import 換成 ，然後只有輸出名稱的字串，以及物件型別。物件型別的格式和引入一樣

#### 注意事項

只能輸出不可變動的全域變數

## 執行架構

在模組載入之後，大部分的物件都已經被儲存到儲存空間裡，或是指定初始值用的物件已經功成身退不需要了，所以剩下這些部份:

* 函式型別 \(types\)
  * 這部份保留參數和回傳值，作為之後檢查型別用
* 輸出物件 \(Exports\)
  * 輸出物件不是儲存在儲存空間，而是儲存在模組，所以這部份完整保留

接下來的位址都是表示這些物件在儲存空間中的位址，因為物件已經載入到儲存空間了，只要留下位址就好

* 函式位址 \(function addresses\)
* 函式表位址 \(function addresses\)
* 記憶體位址 \(function addresses\)
* 全域變數位址 \(function addresses\)

