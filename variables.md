# 變數

變數是用來保存一個數值的物件，依據生存區域的不同，分為**全域變數**和**區域變數**

* 全域變數：在整個程式執行期間都能使用

* 區域變數：伴隨函式產生，只有在該次函式執行的時候能使用 \(下一次呼叫函式的時候，得到的是清空過的區域變數，不會保存上次的數值\)

### 變數宣告

變數在使用前必須要先宣告，讓機器認得那個變數，並為那個變數分配適當的儲存空間。

#### 全域變數宣告

* 不可變動的全域變數
  型別可以是 i32, i64, f32, f64，後面是借用常數指令來設定這個變數一開始的數值，除了對應的 i32.const, i64.const, f32.const, f64.const 之外，不可以用其他指令
* ```
  (module
    (global i32 i32.const 5)
  )
  ```

  這種方式宣告的全域變數不可變動，一旦宣告之後，就不能再改成其他數值

* 可變動的全域變數

* ```
  (module
    (global (mut i32) i32.const 5)
  )
  ```

  加上 mut 表示可變動 \(mutable\)，這樣就能在之後用 set\_global 指令更改這個變數的值

也可以加上以 $ 開頭的名稱，讓之後的存取更方便

```
  (module
    (global $blah (mut i32) i32.const 5)
  )
```

每個全域變數會有一個從 0 開始的編號，因此在 module 裡從上到下分別是 0, 1, 2 ...，之後存取變數的時候就是用編號來區分

#### 區域變數宣告

區域變數是伴隨函式產生，所以區域變數的宣告也是寫在函式的地方

```
(module
  (func (local i32 f32 i32)
    i32.const 5
    drop
  )
)
```

每個區域變數會有一個從 0 開始的編號，因此從左到右分別是 0, 1, 2 ...，之後存取變數的時候就是用編號來區分

當然為了方便，也可以將變數加上以 $ 開頭的名稱，不過就必須分開寫

```
(module
  (func (local $aaa i32) (local $bbb f32) (local i32)
    i32.const 5
    drop
  )
)
```

另外區域變數都是可以變動的，沒有區分可不可變動，使用上要小心不要動到不希望變動的區域變數

#### 變數指令

* **get_global**

  後面接上編號或名稱，將 **全域變數** 的值放進堆疊裡
  ```
  (module
    (global i32 i32.const 4)
    (func
      get_global 0
      unreachable
    )
  )
  ```

* **set_global**

  後面接上編號或名稱，從堆疊裡取出一個數值，放進 **全域變數** 

  **必須是 可變動 的全域變數才能執行這個操作**

  ```
  (module
    (global (mut i32) i32.const 4)
    (func
      get_global 0
      unreachable
      i32.const 5
      set_global 0
      get_global 0
      unreachable
    )
  )
  ```

* **get_local**

  後面接上編號或名稱，將 **區域變數** 的值放進堆疊裡

* **set_local**

  後面接上編號或名稱，從堆疊裡取出一個數值，放進 **區域變數** 

  ```
  (module
    (func (local i32)
      i32.const 7
      i32.const 6
      set_local 0
      get_local 0
      unreachable
    )
  )
  ```

* **tee_local**

  後面接上編號或名稱，從堆疊裡取出一個數值，放進 **區域變數**，不過數值會再放回堆疊裡

  ```
  (module
    (func (local i32)
      i32.const 7
      i32.const 6
      tee_local 0
      get_local 0
      unreachable
    )
  )
  ```

  可以比較一下這個範例和 set_local 範例的不同