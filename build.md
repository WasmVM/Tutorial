# 下載和編譯 WasmVM

WasmVM 是透過 CMake 完成編譯相關的動作和設定。除此之外只有 C++ 標準函式庫，所以編譯的設定上很單純

* 如果沒有 git 、cmake 、 make 的話，請先打下面的指令安裝

```
sudo apt install git cmake binutil
```

* 接著從 GitHub 取得 WasmVM

```
git clone git@github.com:LuisHsu/WasmVM.git
```

* 在 WasmVM 資料夾裡建立 build 資料夾，並進入 build 資料夾

```bash
cd WasmVM && mkdir build && cd build
```

* 執行 CMake 

```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
```

這邊我們使用 Debug 模式，並關閉系統呼叫，讓 WasmVM 可以用比較簡單的方式檢查執行結果

* 執行 Make 編譯程式

```
make
```

等待編譯完成，就可以得到 WasmVM

