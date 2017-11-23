# 算術、參數和控制指令

討論完堆疊和數值形別之後，接下來就能透過指令做簡單的運算

回到 [第一個 WebAssembly 程式](/getting-start.md)，你會看到下面這段程式碼

```
(module
    (func $main
        i32.const 3
        unreachable
    )
    (start $main)
)
```

你可以像下面一樣，用 `;;`** **在程式裡做註解，在`;;`之後，一直到換行為止的文字都會被省略

```
(module
    (func $main
        ;; lalala
        i32.const 3 ;; lalala
        unreachable
    )
    (start $main)
)
```

如果想要一次省略多行，可以用 `(;`** **和`;)`夾住，中間的文字都會被省略

```
(module
    (func $main
        (; woolala
         ;)
        i32.const (; lalala ;) 3
        unreachable
    )
    (start $main)
)
```

---

## 算術指令

### 常數宣告

* `i32.const 整數`
  這個指令會把一個 32 位元的整數放進堆疊
* `i64.const 整數`
  這個指令會把一個 64 位元的整數放進堆疊
* `f32.const 小數` 
  這個指令會把一個單精度浮點數放進堆疊
* `f64.const 小數`
  這個指令會把一個雙精度浮點數放進堆疊

在先前的範例中我們是用 10 進位的數字輸入整數，你也可以在整數或小數的數字前面加上 `0x`，表示輸入的是 16 進位的數字

或是在浮點數的指令裡用 `inf` 輸入無限，`nan` 輸入 NaN

```
(module
    (func $main
        i32.const 3
        i64.const 0x14
        f32.const -0.25
        f64.const -0x2.1
        f32.const -inf
        f32.const nan
        unreachable
    )
    (start $main)
)
```

執行之後得到以下的結果

```
Values in the stack:
Type: f32, Value: nan
Type: f32, Value: -inf
Type: f64, Value: -32.0625
Type: f32, Value: -0.25
Type: i64, Value: 20
Type: i32, Value: 3
```

你可以發現比較晚輸入的數會比較早被列出來，符合堆疊 **後進先出** 的特性

浮點數除了小數之外，也可以用科學記號的方式表示

* 10 進位：$$1.08\times10^{-2} \Rightarrow$$ 1.08e-2 或 1.08E-2
* 16 進位：$$1.08\times16^{-2} \Rightarrow$$ 0x1.08p-2 或 0x1.08P-2

```
(module
    (func $main
        f32.const 1.08e-2
        f32.const 0x1.08p-2
        unreachable
    )
    (start $main)
)
```

```
Values in the stack:
Type: f32, Value: 0.257812
Type: f32, Value: 0.0108
```

### 整數一元運算

一元運算只接受一個數值，因此在堆疊中就是單純的把一個數拿出來，運算完再把結果放回去。

* i32.clz
  * 計算這個整數的位元表示中，最左邊的 1 的左邊有幾個 0
* i32.ctz
  * 計算這個整數的位元表示中，最右邊的 1 的右邊有幾個 0
* i32.popcnt
  * 計算這個整數的位元表示中，總共有幾個 1
* i32.eqz
  * 檢查這個整數是否為 0，是的話放入 1，不是的話放入 0

以下是 i64 的版本，運算方式和 i32 一樣

* i64.clz
* i64.ctz
* i64.eqz
* i64.popcnt

```
(module
    (func $main
        i32.const 2248752  ;; 00000000 00100010 01010000 00110000
        i32.clz 
        unreachable
        i32.const 2248752
        i32.ctz 
        unreachable
        i32.const 2248752
        i32.popcnt 
        unreachable
        i32.const 2248752
        i32.eqz
        unreachable
        i32.const 0
        i32.eqz
        unreachable
    )
    (start $main)
)
```

```
Values in the stack:
Type: i32, Value: 10
Values in the stack:
Type: i32, Value: 4
Values in the stack:
Type: i32, Value: 6
Values in the stack:
Type: i32, Value: 0
Values in the stack:
Type: i32, Value: 1
```

### 整數二元運算

二元運算接受兩個數值，因此在堆疊中把兩個數拿出來，運算完再放回去。

雖然堆疊有後進先出的性質，不過 WebAssembly 為了讓程式比較直覺，在二元指令運算的時候會把順序調換，所以先進堆疊的會放在運算的左邊，後進的會放在右邊，和從堆疊出來的順序相反

下面如果有提到 a, b 兩數，a 表示兩個數值中比較早進入堆疊的數，b 表示比較晚進入堆疊的數

#### 四則運算

* i32.add
  * 兩數相加
* i32.sub
  * 兩數相減
* i32.mul
  * 兩數相乘
* i32.div\_s
  * 兩數當作**有號整數**相除，取商數，捨去小數部份
* i32.div\_u
  * 兩數當作**無號整數**相除，取商數，捨去小數部份
* i32.rem\_s
  * 兩數當作**有號整數**相除，取餘數
* i32.rem\_u
  * 兩數當作**無號整數**相除，取餘數
* 以上指令都有 i64 版本

```
(module
    (func $main
        i32.const 16
        i32.const 11
        i32.add
        unreachable
        i32.const 16
        i32.const 11
        i32.sub
        unreachable
    )
    (start $main)
)
```

```
Values in the stack:
Type: i32, Value: 27
Values in the stack:
Type: i32, Value: 5
```

#### 位元運算

為了方便講解，以下是以 8 位元的整數來舉例

* **i32.and**
  * AND \(或\) 運算：兩個位元都是 1 才會輸出 1，否則輸出 0
  * 把兩個數的位元對齊之後，每個位數分別做 AND 運算

![](/images/and.png)

* **i32.or**
  * OR \(且\) 運算：兩個位元都是 0 才會輸出 0，否則輸出 1
  * 把兩個數的位元對齊之後，每個位數分別做 OR 運算

![](/images/or.png)

* **i32.xor**
  * XOR 運算：兩個位元相同的話輸出 0，否則輸出 1
  * 把兩個數的位元對齊之後，每個位數分別做 XOR 運算

![](/images/xor.png)

* **i32.shl**
  * 把 a 數的位元向左移 b 位，並在右邊補 0

![](/images/shl.png)

* **i32.shr\_u**
  * 把 a 數的位元向右移 b 位，並在右邊補 0

![](/images/rsh_u.png)

* **i32.shr\_s**
  * 把 a 數的位元向右移 b 位，如果 a 數的 sign 是 0 的話補 0，1 的話補 1

![](/images/rsh_s.png)

* **i32.rotl**
  * 把 a 數的位元向左移 b 位，然後把超出去的位元補到右邊

![](/images/lrot.png)

* **i32.rotr**
  * 把 a 數的位元向右移 b 位，然後把超出去的位元補到左邊

![](/images/rrot.png)

* 以上指令都有 i64 版本

#### 比較運算

* **i32.eq**
  * 如果兩數相等輸出 1，否則輸出 0
* **i32.ne**
  * 如果兩數相等輸出 0，否則輸出 1
* **i32.lt\_s**
  * 把兩數當作**有號整數**比較，如果 $$a < b$$輸出 1，否則輸出 0
* **i32.le\_s**
  * 把兩數當作**有號整數**比較，如果 $$a \le b$$ 輸出 1，否則輸出 0
* **i32.lt\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \lt b$$ 輸出 1，否則輸出 0
* **i32.le\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \le b$$ 輸出 1，否則輸出 0
* **i32.gt\_s**
  * 把兩數當作**有號整數**比較，如果 $$a \gt b$$ 輸出 1，否則輸出 0
* **i32.ge\_s**
  * 把兩數當作**有號整數**比較，如果 $$a \ge b$$ 輸出 1，否則輸出 0
* **i32.gt\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \gt b$$ 輸出 1，否則輸出 0
* **i32.ge\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \ge b$$ 輸出 1，否則輸出 0
* 以上指令都有 i64 版本

### 浮點數一元運算

這些操作不會轉換型別，所以就算是得到整數，那個整數的型別一樣是 f32 或 f64

* f32.abs
  * 取絕對值
* f32.neg
  * 正數變負數，負數變正數
* f32.ceil
  * 取大於等於那個數字的最小整數
* f32.floor
  * 取小於等於那個數字的最大整數
* f32.trunc
  * 捨去小數部份，留下整數
* f32.nearest

  * 如果小數部份$$ \lt 0.5$$，捨去小數；如果小數部份$$ \gt 0.5$$  則進位；如果小數部份$$ = 0.5$$，取相鄰整數中是**偶數**的數

  * 乍看之下和四捨五入很像，不過在 "五" 的時候是取偶數，所以 22.5 會得到 22，23.5 會得到 24

* f32.sqrt

  * 開平方根

* 以上指令都有 f64 版本

### 浮點數二元運算

下面如果有提到 a, b 兩數，a 表示兩個數值中比較早進入堆疊的數，b 表示比較晚進入堆疊的數

這些操作不會轉換型別，所以就算是得到整數，那個整數的型別一樣是 f32 或 f64

* f32.add
  * 兩數相加
* f32.sub
  * 兩數相減
* f32.mul
  * 兩數相乘
* f32.div
  * 兩數相除
* f32.eq
  * 如果兩數相等輸出 1，否則輸出 0
* f32.ne
  * 如果兩數相等輸出 0，否則輸出 1
* f32.lt
  * 如果 $$a < b$$ 輸出 1，否則輸出 0
* f32.le
  * 如果 $$a \le b$$ 輸出 1，否則輸出 0
* f32.gt
  * 如果 $$a > b$$ 輸出 1，否則輸出 0
* f32.ge
  * 如果 $$a \ge b$$ 輸出 1，否則輸出 0
* f32.copysign
  * 讓 a 的正負號等於 b 的正負號
* f32.min
  * 取兩數之中比較小的數
* f32.max
  * 取兩數之中比較大的數

### 型別轉換運算

這一系列的操作都是一元運算 \(只接受一個數值\)。和前面的指令不同，這些指令在運算後會把數值的型別轉換成另一種特定的型別

#### Extend

* **i64.extend\_s/i32**
  * 把 i32 轉換成 i64，多出來的部份填上原本**有號整數**的 sign 位元

![](/images/extend.png)

* **i64.extend\_u/i32**
  * 把 i32 轉換成 i64，多出來的部份填上 0

#### Wrap

* **i32.wrap/i64**
  * 把 i64 轉換成 i32，多出來的部份直接捨去

#### Truncate

Truncate 運算是把浮點數根據實際數值轉成整數，如果浮點數的數值是 NaN，$$\pm \infty$$，或是超出該整數型別能表示的數，會得到未定義的結果 \(依據不同的機器、作業系統或執行時期的狀況，可能得到不一樣的結果\)

* **i32.trunc\_s/f32**
  * 將 f32 轉換成 i32 **有**號整數
* **i32.trunc\_s/f64**
  * 將 f64 轉換成 i32 **有**號整數
* **i32.trunc\_u/f32**
  * 將 f32 轉換成 i32 **無**號整數
* **i32.trunc\_u/f64**
  * 將 f64 轉換成 i32 **無**號整數
* **i64.trunc\_s/f32**
  * 將 f32 轉換成 i64 **有**號整數
* **i64.trunc\_s/f64**
  * 將 f64 轉換成 i64 **有**號整數
* **i64.trunc\_u/f32**
  * 將 f32 轉換成 i64 **無**號整數
* **i64.trunc\_u/f64**
  * 將 f64 轉換成 i64 **無**號整數

#### Promote / Demote

* f64.promote/f32
  * 將 f32 依據實際數值轉換成 f64
* f32.demote/f64
  * 將 f64 依據實際數值轉換成 f32

#### Convert

* f32.convert\_s/i32
  * 將 i32 **有**號整數轉換成 f32 
* f32.convert\_s/i64
  * 將 i64 **有**號整數轉換成 f32
* f32.convert\_u/i32
  * 將 i32 **無**號整數轉換成 f32
* f32.convert\_u/i64
  * 將 i64 **無**號整數轉換成 f32
* f64.convert\_s/i32
  * 將 i32 **有**號整數轉換成 f64 
* f64.convert\_s/i64
  * 將 i64 **有**號整數轉換成 f64
* f64.convert\_u/i32
  * 將 i64 **無**號整數轉換成 f64
* f64.convert\_u/i64
  * 將 i64 **無**號整數轉換成 f46

#### Reinterpret

Reinterpret 比較特別，是針對相同位元長度的整數或浮點數，在位元不變的情況下，重新把整數用浮點數的位元格式詮釋成浮點數，或把浮點數用整數的位元格式詮釋成整數

* i32.reinterpret/f32
  * 將 f32 重新詮釋成 i32
* i64.reinterpret/f64
  * 將 f64 重新詮釋成 i64
* f32.reinterpret/i32
  * 將 i32 重新詮釋成 f32
* f64.reinterpret/i64
  * 將 i64 重新詮釋成 f64

---

## 參數指令

* drop
  * 從堆疊中拿出一個數值，然後捨棄不用
* select
  * 從堆疊中拿出 3 個數值，這邊假設為 a, b, c
    * 如果 $$c \ne 0$$ ，把 a 放回堆疊
    * 如果 $$c = 0$$ ，把 b 放回堆疊

---

## 控制指令

* **nop**
  * 不做任何事
* **unreachable**
  * 這個指令的本意是製造一個例外狀況，不過在 WasmVM 裡利用製造的中斷來實作系統呼叫
  * 在有開啟系統呼叫時，unreachable 會執行系統呼叫
  * 沒開啟系統呼叫時
    * 以 Debug 模式編譯，會輸出堆疊裡的數值，不放回堆疊，方便除錯
    * 以 Release 模式編譯，會中止程式，並得到錯誤訊息

### 區塊

接下來的 block、loop、if 指令會開啟新的程式區塊 \(block\)。其實就是在堆疊裡放入一個標籤 \(Label\)，這個 label 會記錄目前所處的函式、進入 block 時程式執行的位置，以及離開 block 的時候要接著執行的位置。

等到要離開 block 的時候，再從堆疊把剛剛放進去的 label 拿出來，根據之前的記錄決定要從哪邊繼續執行程式

大部分的組合語言會在程式的某個部份加入 label ，這種 label 就只是標籤而已，標示可以給 jump 指令跳過去的地方，再用 jump 指令改變程式的執行順序

![](/images/jump.png)

然而這種 label 可以放在幾乎任何地方，當 label 和 jump 指令變多，程式的執行順序會不好掌握，容易造成問題

WebAssembly 的 block 比較像是開一個新的空間，在新的空間裡執行 block 裡的指令。一層層的架構形成結構化的控制流程 \(Structured Control Flow\)，讓指令的執行時機比較好掌控。

![](/images/block.png)

其實 block 也不是完全開一個全新的空間，而是在堆疊裡的 label 有類似"遮罩"的作用，把堆疊裡的數值先遮住，讓後面的程式看不到先前留在堆疊裡的數值，block 結束之後 label 被拿走，原本在堆疊裡的數值又重見天日

block 在結束的時候如果沒有指定回傳值，必須要把堆疊裡剩下的數值用 drop 捨棄或是用其他方法清空，否則在轉換成 wasm 時會產生錯誤而無法轉換

以下是會產生區塊的控制指令 \(函式也會，不過留待 [函式](/function.md) 章節再做討論\)

* **block ... end**

  * block 和 end 是成對存在，中間放入要在 block 裡執行的指令，可以參考以下範例
  * ```
    (module
        (func $main
            block
                i32.const 5
                unreachable
            end
        )
        (start $main)
    )
    ```
  * 也可以幫 block 加上一個開頭是 $ 的名稱，方便之後的 br 指令操作

  * ```
    (module
        (func $main
            block $aaa
                i32.const 5
                unreachable
            end
        )
        (start $main)
    )
    ```
  * 還可以指定一個 block 的回傳值，這樣 block 在結束的時候，就不一定要清空，可以在堆疊裡留下一個數值給上一層的 block 使用
  * ```
    (module
        (func $main
            block $aaa (result i32)
                i32.const 5
            end
            unreachable
        )
        (start $main)
    )
    ```

* **loop ... end**

  * loop 和 block 大致上相同，唯一不同的地方在於 block 執行完之後會繼續執行接下來的指令，loop 則是再回到 loop 的開頭執行 loop 裡的指令
  * ```
    (module
        (func $main
            loop
                i32.const 3
                unreachable
            end 
        )
        (start $main)
    )
    ```

    上面這段範例執行之後發現停不下來是正常的，因為我們沒有跳出 loop，所以會一直不斷的回到 loop 的地方執行。這時可以按 Ctrl + C 強制結束程式

* **if ... else ... end**

  * if 會從堆疊裡拿出一個 i32 數值，如果這個數值不是 0，開一個 block 執行 if 到 else 之間的指令；如果這個數值是 0，開一個 block 執行 else 到 end 之間的指令
  * ```
    (module
        (func $main
            i32.const 1
            if (result i32)
                i32.const 2
            else
                i32.const 3
            end
            unreachable
        )
        (start $main)
    )
    ```

    執行結果

  * ```
    Values in the stack:
    Type: i32, Value: 2
    ```

    如果把 1 換成 0，會得到以下的結果

  * ```
    Values in the stack:
    Type: i32, Value: 3
    ```
  * 可以省略 else 的部份，這樣當 if 從堆疊拿出 0 的時候就會繼續執行 end 之後的指令，不會執行 if 到 end 之間的指令

  * ```
    (module
        (func $main
            i32.const 1
            if (result i32)
                i32.const 2
            end
            unreachable
        )
        (start $main)
    )
    ```

### 分支指令

Branch instruction 一般會翻譯為 "分支指令"，不過我認為在 WebAssembly 用 "跳回" 的概念會比較好理解。上面有提到 block 是一層一層的概念，跳回 $$n$$ 在 WebAssembly 裡其實就是回到上 $$n+1$$ 層，然後執行接下來的指令，例如：

```
(module
    (func $main
        loop
           i32.const 5
           unreachable
           br 0
        end
        i32.const 3
        unreachable
    )
    (start $main)
)
```

br 0 會回到上 $$0+1$$ 層 \(就是上一層\)，然後接著執行 loop ... end 之後的指令，也就是 i32.const 3

```
(module
    (func $main
        block
            loop
                i32.const 5
                unreachable
                br 1
            end
            i32.const 3
            unreachable
        end
        i32.const 4
        unreachable
    )
    (start $main)
)
```

br 1 會回到上 $$1+1$$ 層，所以是接著執行 block ... end 之後的指令，也就是 i32.const 4

* **br**

  * 無條件跳回，請參考上面的範例
  * 如果 block、loop 或 if 有名稱的話，也可以在 br 後面改用名稱決定要跳出哪一層 block
  * ```
    (module
        (func $main
            block $aaa
                loop
                    i32.const 5
                    unreachable
                    br $aaa
                end
                i32.const 3
                unreachable
            end
            i32.const 4
            unreachable

        )
        (start $main)
    )
    ```

* **br\_if**

  * 從堆疊取出一個 i32 數值，如果該數值是 0，不做任何事；如果該數值不是 0 則執行跳回的動作
  * ```
    (module
        (func $main
            block $aaa
                loop
                    i32.const 5
                    br_if $aaa
                end
                i32.const 3
                unreachable
            end
            i32.const 4
            unreachable

        )
        (start $main)
    )
    ```

* **br\_table**

  * 後面會接著至少一個要跳回的目標，然後從堆疊中取出一個 i32 數值 $$n$$，如果 $$ 0 \le n \lt 總數量$$，跳回第 $$n-1$$ 個目標 \(從 0 開始數\)，否則跳回最後一個目標

  ![](/images/brtable.png)



