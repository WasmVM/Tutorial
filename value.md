# 數值型別

## 位元、位元組

### 位元 \(Bit\)

電腦是透過電來操作，而電路的狀態可以簡單的用 0 \(斷電\) 或 1 \(通電\) 來表示，因此一個表示 0 或 1 的單位我們稱為一個 **位元**

一筆資料裡最大的位元 \(Most Significant Bit\)，習慣上簡稱為 **MSB** ，最小的位元 \(Least Significant Bit\)，習慣上簡稱為 **LSB**

### 位元組 \(Byte\)

習慣上我們會把 8 個位元做為一組，當成一個單位，稱為 **位元組**

### 位元組順序 \(Byte order\)

當一筆資料用不只一個位元組來表示的時候，不同的位元組之間會有誰先誰後的順序問題。像是 1234 這個數字，習慣上我們會從左到右寫成 1234，不過有些中式寫法也可能從右到左寫，變成 4321。因此根據從比較大的位數 \(1\) 開始或從比較小的位數 \(4\) 開始的不同，可以分成以下兩種

* big-endian

  比較大的位元組 -&gt; 比較小的位元組，在日常生活中、網路，或是某些作業系統比較常使用

* little-endian

  比較小的位元組 -&gt; 比較大的位元組，在大部份的作業系統比較常使用

假設一個以16進位表示的整數 0x12ABCDEF, 在 big-endian 的系統會排列成 0x12ABCDEF，little-endian 的系統會排列成 0xEFCDAB12

**\* 注意一個 16 進位的數字只有 4 位元，所以一個位元組會有 2 個 16 進位的數字**

WebAssembly 的規範中一律採用 **little-endian** 排列方式

## 整數 \(Integer\)

我們生活中使用的整數是 10 進位整數，然而在電腦中是以 0 和 1 表示各種資料，所以使用 2 進位整數。一筆整數資料中可能會利用一個位元表示正負號，或是沒有正負號，因此分為 **有號整數** 和 **無號整數**

### 無號整數

沒有表示正負號的位元，直接使用整數的二進位表示 $$0 \sim 2^{31} - 1$$ 之間的 **正整數**

### 有號整數

有表示正負號的位元，習慣上會用 MSB 表示正負號，所以能表示$$-2^{31} \sim 2^{31}-1$$之間的整數

#### 1 補數 \(1's Complement\)

電腦上的減法是用"加負數"的方式實作，為了方便運算，在負數的部份我們可以把正數做位元反轉，像是 8 位元的 00000001 \(1\) 取負數之後就變成 11111110 \(-1\)，這種表示負數的方式稱為 **1 補數。**

以 8 位元的 1 - 2 為例，用 1 補數的運算會變成

00000001 \(1\) + 11111101 \(-2\) = 11111110 \(-1\)

很輕易的就用加法完成減法運算

**1 補數的缺點**

1. 有 +0, -0 之分

   0 在數學上是沒有正負之分的，不過對 0 做反轉會得到一長串的 1，這個就稱之為 -0。-0 的出現讓運算時需要再判斷有沒有 -0 存在，增加了複雜度

2. 需要循環進位

   當兩數相加有超出範圍的進位時，需要把超出的進位再加回去，不然會發生錯誤，請看下面的例子

   11111110 \(-1\) + 00000010 \(2\) = ~~1~~00000000 \(0\) 這是錯誤的

   要把捨掉的 1 加回去答案才會變成 00000001 \(1\)

#### 2 補數 \(2's Complement\)

1 補數存在負零和需要循環進位等等缺點，因此現今的電腦普遍採用的是 **2 補數**這種表示方式。

2 補數的負數算法是將位元反轉之後，再把結果加 1

例如 8 位元的 00000001 \(1\)，位元反轉變成 11111110，再加 1 成為 11111111 \(-1\)  
使用 2 補數可以避免負零的產生，因為 0 在位元反轉之後是 11111111，加 1 之後是 ~~1~~ 00000000，把進位捨去一樣是原本的 0。兩數相加之後如果有進位也只需要把多出來的進位捨去即可，沒有需要循環進位的問題

WebAssembly 的規範中一律採用 **2 補數** 作為有號整數的表示方法

## 浮點數 \(Floating point number\)

浮點數也就是小數，不像整數一樣直接用 2 進位，而是有特殊的表示方式。

### 2 進位科學記號

單精度浮點數有 32 位元，以 2 進位科學記號表示

Ex：

* $$0.25_{(十進位)} = 0.01_{(二進位)} = 1.0_{(bin)} \times 2^{-2}$$
* $$0.3125_{(十進位)} = 0.0101_{(二進位)} = 1.01_{(bin)} \times 2^{-2}$$
* $$4.5_{(十進位)} = 100.1_{(二進位)} = 1.001_{(bin)} \times 2^{2}$$

除了 0 之外，其他的數必定有 1 ，要讓小數點左邊只剩下最大的 1

### 單精度浮點數 \(Single-precision\) & 位元格式

浮點數的位元分成以下3個區域

![](.gitbook/assets/float.png)

* 正負號 \(sign\) 0 表示正數，1表示負數
* 指數 \(exponent\)
  * $$2^{指數+127}$$     所以 0.25 的指數部份 \(-2\) 會用 01111101 \(125\) 表示
  * 指數值範圍在 1~254 之間，也就是實際值在 -126 ~ 127 之間，255 保留給特殊值，0 表示非規約形式 \(下面會做說明\)
* 有效數 \(fraction\)
  * 規約形式 \(canonial\) : 當指數部份的實際值大於 -127 \(也就是加 127 之後大於 0\)，把小數點左邊的 1 捨掉，取小數點後的部份靠左對齊
  * 非規約形式 \(non - canonial\) : 當指數部份的實際值小於或等於 -127，也就是相當接近 0 的數。這時候我們不再把小數點前留下一個 1 ，而是設法讓指數的實際值變成 -126，再取小數點後的部份靠左對齊
    * 指數部份是 0。雖然實際值應該會是 -127，但是在非規約形式下因為要設法讓他變成 -126，所以其實是 -126
* 特殊值

  | 數值 | Sign | Exponent | Fraction |
  | :---: | :---: | :---: | :---: |
  | $$+ 0$$ | 0 | 全 0 | 全 0 |
  | $$-0$$ | 1 | 全 0 | 全 0 |
  | $$+ \infty$$ | 0 | 全 1 | 全 0 |
  | $$-\infty$$ | 1 | 全 1 | 全 0 |
  | NaN 未定義\(Not a Number\) | 1 或 0 | 全 1 | 不是全都 0 |

* 範例
  * 0.15625 $$(1.01_{(bin)} \times 2^{-3})$$
    * Sign : 正數$$\Rightarrow$$ 0
    * Exponent : $$-3+127 = 124 \Rightarrow 01111100$$
    * Fraction : $$1.01 - 1 = 0.01 \Rightarrow 010000 \ldots 0$$
  * $$-1.010_{(bin)} \times 2^{-128}$$
    * Sign : 負數$$\Rightarrow 1$$
    * Exponent : 0 \(non - canonial\)
    * Fraction : $$1.010_{(bin)} \times 2^{-128} = 0.0101_{(bin)} \times 2^{-126} \Rightarrow 01010000 \ldots 0$$

### 雙精度浮點數 \(Double-precision\)

![](.gitbook/assets/double%20%281%29.svg)

和單精度的表示法一樣，不過 Exponent 有 11 位元，Fraction 有 52 位元

因此在 Exponent 的部份，變成 $$2^{指數+1023}$$，指數值範圍在 1 ~ 2046，0 一樣是非規約形式，2047 是特殊值

特殊值的定義也和單精度一樣

### 浮點數誤差

有蠻多小數無法用二進位整除，像是 0.3、0.7 ......所以得到的其實是非常接近的近似值

有些電腦或編譯器會針對誤差做一部份的修正，所以不一定會出錯，但是編寫程式的時候還是要注意誤差的問題

## 堆疊裡的數值型別

堆疊裡的數值型別有以下 4 種

1. i32 : 32 位元整數
2. i64 : 64 位元整數
3. f32 : 32 位元\(單精度\)浮點數
4. f64 : 64 位元\(單精度\)浮點數

在整數方面，無論是有號或無號整數，存進堆疊的時候會保持原來的位元形式，不會特別區分有號或無號。這個區分是依據不同的算術指令來達成不同的效果

