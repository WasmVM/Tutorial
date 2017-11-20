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

* 10 進位：$$$$$$1.08\times10^{-2} \Rightarrow$$ 1.08e-2 或 1.08E-2
* 16 進位：$$$$$$1.08\times16^{-2} \Rightarrow$$ 0x1.08p-2 或 0x1.08P-2

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

一元運算只接受一個數值，因此在堆疊中就是單純的把一個數拿出來，運算完再放回去。

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

* i32.and
  * AND \(或\) 運算：兩個位元都是 1 才會輸出 1，否則輸出 0
  * 把兩個數的位元對齊之後，每個位數分別做 AND 運算

![](/images/and.png)

* i32.or
  * OR \(且\) 運算：兩個位元都是 0 才會輸出 0，否則輸出 1
  * 把兩個數的位元對齊之後，每個位數分別做 OR 運算

![](/images/or.png)

i32.xor

* XOR 運算：兩個位元相同的話輸出 0，否則輸出 1
* 把兩個數的位元對齊之後，每個位數分別做 XOR 運算

![](/images/xor.png)

* i32.shl
  * 假設那兩個整數為 a, b，把 a 數的位元向左移 b 位，並在右邊補 0

![](/images/shl.png)

* i32.shr\_u
* i32.shr\_s
* i32.rotl
* i32.rotr
* 以上指令都有 i64 版本

#### 比較運算

* i32.eq
* i32.ne
* i32.lt\_s
* i32.le\_s
* i32.lt\_u
* i32.le\_u
* i32.gt\_s
* i32.ge\_s
* i32.gt\_u
* i32.ge\_u
* 以上指令都有 i64 版本

### 浮點數一元運算

### 浮點數二元運算

### 型別轉換

---

## 參數指令

---

## 控制指令



