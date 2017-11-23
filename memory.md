# 記憶體 (Memory)

WebAssembly 的記憶體是以 byte 為單位的連續空間，大小從 0 開始，以 $$64 \times 1024$$ bytes 為單位漸漸擴充，每 $$64 \times 1024$$ bytes 稱為一個 **page**

![](/images/memory.png)

WebAssembly 是設計在瀏覽器執行，安全性上的考量很重要。因此記憶體專門存放資料，指令、變數或其他物件並不會共用同一塊記憶體，避免一些因為資料溢出記憶體而造成的安全性問題

## 宣告記憶體

記憶體在使用前也是要經過宣告，一個模組最多只能有一個記憶體

```
(module
	(memory 1 5)
)
```
第一個數字是最小值，表示一開始的大小

第二個數字是最大值，可加可不加。如果沒有指定最大值，這塊記憶體的成長沒有上限

另外有一種方便的寫法，可以在宣告的同時為記憶體設定初始的資料

```
(module
	(memory (data "Hello\0A\00"))
)
```

我們常見的字母，在電腦通常會用一個 byte 儲存，每個字母都有對應的數值，這個對應的值我們稱為"字碼"。WebAssembly 是使用 Unicode 做為字元編碼，由於 Unicode 編碼的數量龐大，不方便在這邊做詳細說明，想要了解的話請自行上網搜尋

除了 Unicode ，也可以用一個反斜線 \ 後面接兩個 16 進位數字表示一個 byte

### 取得/改變記憶體大小

* **current_memory**
	* 將目前的記憶體大小用 page 為單位，以 i32 數值放入堆疊中
* **grow_memory**
	* 從堆疊中拿出一個 i32 數值 $$ n $$ ，將目前的記憶體大小增加 $$ n page $$ 
		* 如果成功，將舊的大小用 page 為單位，以 i32 數值放入堆疊中
		* 如果失敗，將 -1 以 i32 數值放入堆疊中

	* 失敗的原因可能是在有指定最大值的情況下已經超過最大值，或是硬體空間不足。

### 讀取記憶體

這些指令都會從堆疊中拿出一個 i32 的數值當作記憶體位址

* **i32.load8_s**
	* 從記憶體讀取一個 8 bit 大小的資料，這 8 bit 之外的部份用 **有號整數 extend** 的方式填滿成 32 bit，再以 i32 型別放入堆疊中
* **i32.load8_u**
	* 從記憶體讀取一個 8 bit 大小的資料，這 8 bit 之外的部份用 0 填滿成 32 bit，再以 i32 型別放入堆疊中
* **i32.load16_s**
	* 從記憶體讀取一個 16 bit 大小的資料，這 16 bit 之外的部份用 **有號整數 extend** 的方式填滿成 32 bit，再以 i32 型別放入堆疊中
* **i32.load16_u**
	* 從記憶體讀取一個 16 bit 大小的資料，這 16 bit 之外的部份用 0 填滿成 32 bit，再以 i32 型別放入堆疊中
* **i32.load**
	* 從記憶體讀取一個 32 bit 大小的資料，以 i32 型別放入堆疊中
* **i64.load8_s**
	* 從記憶體讀取一個 8 bit 大小的資料，這 8 bit 之外的部份用 **有號整數 extend** 的方式填滿成 64 bit，再以 i64 型別放入堆疊中
* **i64.load8_u**
	* 從記憶體讀取一個 8 bit 大小的資料，這 8 bit 之外的部份用 0 填滿成 64 bit，再以 i64 型別放入堆疊中
* **i64.load16_s**
	* 從記憶體讀取一個 16 bit 大小的資料，這 16 bit 之外的部份用 **有號整數 extend** 的方式填滿成 64 bit，再以 i64 型別放入堆疊中
* **i64.load16_u**
	* 從記憶體讀取一個 16 bit 大小的資料，這 16 bit 之外的部份用 0 填滿成 64 bit，再以 i64 型別放入堆疊中
* **i64.load32_s**
	* 從記憶體讀取一個 32 bit 大小的資料，這 32 bit 之外的部份用 **有號整數 extend** 的方式填滿成 64 bit，再以 i64 型別放入堆疊中
* **i64.load32_u**
	* 從記憶體讀取一個 32 bit 大小的資料，這 32 bit 之外的部份用 0 填滿成 64 bit，再以 i64 型別放入堆疊中
* **i64.load**
	* 從記憶體讀取一個 64 bit 大小的資料，以 i64 型別放入堆疊中
* **f32.load**
	* 從記憶體讀取一個 32 bit 大小的資料，以 f32 型別放入堆疊中
* **f64.load**
	* 從記憶體讀取一個 64 bit 大小的資料，以 f64 型別放入堆疊中

範例

```
(module
	(memory (data "\0A\01\00\00\00"))
	(func $main
		i32.const 0
		i32.load
		unreachable
	)
	(start $main)
)
```

執行結果

```
Values in the stack:
Type: i32, Value: 266
```

### 寫入記憶體

這些指令都會從堆疊中拿出一個 i32 的數值當作記憶體位址，再拿出一個相對應型別的數值

* **i32.store8**
	* 將 i32 數值寫入 8 bit 大小的資料到記憶體中，超出的部份捨棄
* **i32.store16**
	* 將 i32 數值寫入 16 bit 大小的資料到記憶體中，超出的部份捨棄
* **i32.store**
	* 將 i32 數值寫入到記憶體中
* **i64.store8**
	* 將 i64 數值寫入 8 bit 大小的資料到記憶體中，超出的部份捨棄
* **i64.store16**
	* 將 i64 數值寫入 16 bit 大小的資料到記憶體中，超出的部份捨棄
* **i64.store32**
	* 將 i64 數值寫入 32 bit 大小的資料到記憶體中，超出的部份捨棄
* **i64.store**
	* 將 i64 數值寫入到記憶體中
* **f32.store**
	* 將 f32 數值寫入到記憶體中
* **f64.store**
	* 將 f64 數值寫入到記憶體中

範例

```
(module
	(memory (data "\0A\01\00\00\00"))
	(func $main
		i32.const 0
		i32.load
		unreachable
		i32.const 1
		f32.const 1.5
		f32.store
		i32.const 0
		f32.load
		unreachable
	)
	(start $main)
)
```

執行結果

```
Values in the stack:
Type: i32, Value: 266
Values in the stack:
Type: f32, Value: -2

```

另外也可以在指令後面指定 offset，表示要把記憶體位置再額外位移幾個 byte

```
(module
	(memory (data "\0A\01\00\00\00"))
	(func $main
		i32.const 0
		i32.load offset=1
		unreachable
	)
	(start $main)
)
```

執行結果

```
Values in the stack:
Type: i32, Value: 1
```

如果有看 WebAssembly 規格的朋友應該還會多知道一個 align，不過在規格上這個 align 是在優化時作為提示用，不是必要，所以 WasmVM 裡目前並不會使用到