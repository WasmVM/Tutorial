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

你可以像下面一樣，用 **`;;` **在程式裡做註解，在`;;`之後，一直到換行為止的文字都會被省略

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

## 算術指令

### 常數宣告

* `i32.const 整數`
  這個指令會把一個 32 位元的整數放進堆疊
* `i64.const 整數`
  這個指令會把一個 64 位元的整數放進堆疊
* `f32.const 小數` 
  這個指令會把一個 32 位元的小數放進堆疊
* `f64.const 小數`
  這個指令會把一個 64 位元的小數放進堆疊



### 整數一元運算

### 整數二元運算

### 浮點數一元運算

### 浮點數二元運算

### 型別轉換

## 參數指令

## 控制指令

