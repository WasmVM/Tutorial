# 函式 (Function)

函式是一段執行特定功能的指令集合，WebAssembly 的函式可以傳入若干個數值當作區域變數、定義屬於函式自己的區域變數，以及在結束時傳回一個數值。尤其在瀏覽器上更是 WebAssembly 和網頁的 JavaScript 程式碼互動的重要角色

函式的運作機制和 block 差不多，會在堆疊裡放進一個框架 \(Frame\)，這個 frame 會記錄目前在哪個模組，以及函式的區域變數。同樣的，frame 會像遮罩一樣把先前的數值和 label 遮住，讓函式裏面的指令看不到先前的數值和 label，等到函式結束的時候再把 frame 取出，回到先前的狀態

函式執行的時候自帶一個 block，所以也會有和 block 差不多的行為，結束時一樣要把堆疊清空，或是在有指定回傳值的情況下剩下一個相對應的回傳值

## 宣告函式

函式的宣告非常簡單，像全域變數和記憶體一樣宣告在模組裡，裏面放入要在函式裡執行的指令

```
(module
	(func
		i32.const 3
		drop
	)
	(start 0)
)
```

可以幫函式加上名稱，像之前的範例就有幫函式加上 $main

```
(module
	(func $main
		i32.const 3
		drop
	)
	(start $main)
)
```

函式和全域變數一樣會有一個從 0 開始的編號，因此在 module 裡從上到下分別是 0, 1, 2 ...，之後呼叫函式的時候也是用編號來區分

## 呼叫函式

* **call**
	* 後面接編號。呼叫之後會執行被呼叫的函式，直到被呼叫的函式返回之後，才會繼續執行接下來的指令

```
(module
	(func 
		i32.const 2
		unreachable
	)
	(func $main 
		call 0
		i32.const 3
		unreachable
	)
	(start $main)
)
```

## 參數 (Parameter)

如果要傳遞一些數值到函式裡，可以用 param 加上參數，參數會被當作函式的區域變數

**有寫在 start 裡的函式不能加參數**，在[模組](module.md)章節會有詳細說明

```
(module
	(func $aaa (param i32 f32 i32) 
		get_local 0
		get_local 2
		i32.add
		get_local 1
		unreachable
	)
	(func $main 
		i32.const 5
		f32.const 3.14
		i32.const 8
		call $aaa
		i32.const 3
		unreachable
	)
	(start $main)
)
```

執行結果

```
Values in the stack:
Type: f32, Value: 3.14
Type: i32, Value: 13
Values in the stack:
Type: i32, Value: 3
```

和區域變數一樣，可以幫參數加上名稱，不過也是一樣要分開寫

```
(module
	(func $aaa (param $a i32) (param f32) (param $b i32)
		get_local $a
		get_local $b
		i32.add
		get_local 1
		unreachable
	)
	(func $main 
		i32.const 5
		f32.const 3.14
		i32.const 8
		call $aaa
		i32.const 3
		unreachable
	)
	(start $main)
)
```

同時使用 param 和 local 時，local 必須在 param 之後，當然區域變數的編號也會排在 param 的後面

```
(module
	(func $aaa (param $a i32) (local $b i32)
		get_local $a
		i32.const 8
		set_local 1
		get_local $b
		i32.add
		unreachable
	)
	(func $main 
		i32.const 5
		call $aaa
		i32.const 3
		unreachable
	)
	(start $main)
)
```

## 回傳值 (Return value)

如果要在函式結束時傳遞數值給上一層函式，可以用 result 加上一個回傳值。這樣在函式結束的時候會從堆疊拿出一個對應的數值，在回到上一層函數時放回堆疊裡

**有寫在 start 裡的函式不能加回傳值**，在[模組](module.md)章節會有詳細說明

```
(module
	(func $add (param $a i32) (param $b i32) (result i32)
		get_local $a
		get_local $b
		i32.add
	)
	(func $main 
		i32.const 5
		i32.const 4
		call $add
		unreachable
	)
	(start $main)
)
```

執行結果

```
Values in the stack:
Type: i32, Value: 9
```

使用 param、result 和 local 時，必須按照 param $$ \rightarrow $$ result $$ \rightarrow $$ local 的順序排列