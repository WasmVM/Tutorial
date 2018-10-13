# 第一個 WebAssembly 程式

## Wabt 工具

WebAssembly \(之後都簡稱 Wasm\) 程式主要是儲存成位元格式 \(Binary format\)，對人類來說比較不方便閱讀。

然而 Wasm 也有制定比較方便閱讀的文字格式 \(Text format\)

但是文字格式需要轉成位元格式，WasmVM才有辦法執行

WebAssembly官方有提供 Wabt 工具可以轉換格式，以下有幾種使用方式：

* **直接在網站上使用 Wabt**

網址: [https://cdn.rawgit.com/WebAssembly/wabt/fb986fbd/demo/wat2wasm](https://cdn.rawgit.com/WebAssembly/wabt/fb986fbd/demo/wat2wasm/]%28https://cdn.rawgit.com/WebAssembly/wabt/fb986fbd/demo/wat2wasm/\)

* **下載 Wabt 的離線版網站**

在終端機執行

```text
git clone git@github.com:WebAssembly/wabt.git
```

用瀏覽器打開資料夾裡的 demo/index.html

* **下載並編譯、執行 wat2wasm 程式**

從 GitHub 下載 Wabt

```text
git clone git@github.com:WebAssembly/wabt.git
```

建立 build 資料夾並執行 CMake

```text
cd wabt && mkdir build && cd build && cmake -DBUILD_TESTS=OFF ..
```

執行 Make

```text
make
```

你會得到 wat2wasm 程式，在你用文字編輯器編輯完文字格式的檔案後，執行

```text
./wat2wasm 文字檔名 -o 要輸出的位元格式檔名 -v
```

這邊用 -v 是為了在教學中能方便查看位元檔內容，如果你已經不需要查看內容，可以不用打 -v

## 編寫 Wasm 程式

請把網頁版的左上角清空，輸入以下程式碼。 用 wat2wasm 的話請用文字編輯器編輯文字檔

```text
(module
    (func $main
        i32.const 3
        unreachable
    )
    (start $main)
)
```

成功的話會看到以下訊息

```text
0000000: 0061 736d                                 ; WASM_BINARY_MAGIC
0000004: 0100 0000                                 ; WASM_BINARY_VERSION
...以下省略
```

按 Download 下載檔案或使用 wat2wasm 產生檔案之後，用 WasmVM 執行

```text
./WasmVM 檔案名稱
```

成功的話會看到以下訊息

```text
Values in the stack:
Type: i32, Value: 3
```

如果看到以下訊息，表示之前編譯的時候系統呼叫功能關了，但是沒有用 Debug 模式

```text
(func: 0, offset: 3) [unreachable] Trap without syscall provided.
```

如果看到以下訊息，表示之前編譯的時候系統呼叫功能沒有關

```text
(func: 0, offset: 3) [syscall][sys_close] No enough value in the stack.
```

發生以上兩種狀況，請先執行

`make clean`

然後重新執行 CMake 和 Make 編譯程式

