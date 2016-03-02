目前有支援的選項設定

!

改寫，設定這個選項，你可以改寫之前定義的程式片段，預設是列出清單，來選擇。

b

每行起始位置-當設定了這個選項時，啟動字只有在一行中的第一個字，才能擴展成為程式片段，即便啟動字前有任何的空白符號 ，也能擴展。
一般預設狀況是，啟動字在任何位置上只要前面是空白字符，皆會擴展。

i

在字之中-一般預設只要在一行的第一個字或是前面有一個空白或多個空白就可以擴展。一旦設定這個選項，就不會考慮前面是什麼字元，即使在啟動字在任何字之中。

w

字的邊界-一般而言，啟動字是由字母數字以及'_'所組成（詳細定義請找iskeyword設定），所以一旦這個選項設定之後，啟動字的邊界只要不是上述的字符，即可被擴展，換句話說，啟動字前面不可以有上述的字符。如果這個啟動字是一個字的前半部，此時並不會把後綴部分一起擴展。

r

正規表示式-當設定這個選項後，啟動字必須為正規表示式，且須用雙引號刮起來，當輸入的字元符合啟動字的定義時，就會擴展，而且符合的這些字元會送到程式片段中的一個 python 區域變數 match。

```
snippet "be(gin)?( (\S+))?" "begin{} / end{}" br
\begin{${1:`!p
snip.rv = match.group(3) if match.group(2) is not None else "something"`}}
    ${2:${VISUAL}}
\end{$1}$0
endsnippet
```

t

不改變tab格式-如果程式片段的縮排是使用tab的話，預設擴展的話vim會根據設定('shiftwidth', 'softtabstop', 'expandtab', 'tabstop')，來插入縮排格式，舉例來說，當設定了expandtab，那麼tab將會被空白字符取代。所以套用這個選項時，插入的tab格式會跟程式片段中的tab格式一致。

s

在跳換編輯程式片段的過程中，會立即將行尾有空白字元的部分刪除。
舉例來說，當不填${2}時，在跳到${3}之前會把前面的空白符刪除。

```
snippet sst 
  \start${1} ${2}
    ${3} 
  \stop${1/\s*//g} 
  $0 
endsnippet
```

vim 7.4

```
   m   Trim all whitespaces from right side of snippet lines. Useful when
       snippet contains empty lines which should remain empty after expanding.
       Without this option empty lines in snippets definition will have
       indentation too.

   e   Context snippets - With this option expansion of snippet can be
       controlled not only by previous characters in line, but by any given
       python expression. This option can be specified along with other
       options, like 'b'. See |UltiSnips-context-snippets| for more info.

   A   Snippet will be triggered automatically, when condition matches.
       See |UltiSnips-autotrigger| for more info.
       
```
∬

## 4.7 轉換

注意：這個章節比較難以理解，所以分為兩個部分，第一部份談到轉換的相關語法，第二部分則是一些設計實際的轉換操作。

轉換很像一種映射，但是不只是逐字的複製停頓點所輸入的文字而已，轉換會根據正規表示式符合的結果來進行套用。
在 UltiSnips 中功能跟語法上都是非常接近 TextMate 。

轉換語法

```
${<tab_stop_no/regular_expression/replacement/options}
```

每個部份的定義如下

```
   tab_stop_no        - 參考第幾個停頓點
   regular_expression - 正規表示式，用來檢測停頓點所輸入的字串是否符合
   replacement        - 取代文字，細部說明如下
   options            - 正規表示式的選項
```

正規表示式的選項可以是以下的組合

```
   g    - 全域取代
          預設上，只會取代第一個符合正規表示式的結果，當設定這個選項，所有符合的結果都會被取代。
   i    - 不區分大小寫
          預設上，正規表示式是會區分大小寫，加上這個設定，即可忽略大小寫問題。
   a    - ascii 轉換
          預設上，轉換動作都是在 utf-8 字串上進行，加上這個選項，所有符合的結果都會轉換對應到 ASCII 字串上，
          例如 'à' 會變成 'a' 。
          這個選項需要安裝 python 套件 'unidecode'.
```

正規表示式的語法討論已經超出這份文件所討論的範圍了，事實上這個套件所用到的都是 Python
正規表示式的語法，所以可以將 python 're' module 用來當作學習指南，詳細請看 http://docs.python.org/library/re.html 。

取代字串的語法是非常特別的，下一個段落會有更詳細的描述。

### 4.7.1 取代字串

取代字串中可包含著 $no 變數，例如： $1 ，這個變數是從正規表示式中有相符 group 而得來的，
而 $0 這個變數比較特別，他代表整體相符的結果，取代字串可以包含著些特別的跳脫序列。

```
   \u   - 將所在位置的下一個字元變成大寫
   \l   - 將所在位置的下一個字元變成小寫
   \U   - Uppercase everything till the next \E
   \L   - Lowercase everything till the next \E
   \E   - End upper or lowercase started with \L or \U
   \n   - A newline
   \t   - A literal tab
```

最後，取代字串可以包含著一些條件式的取代字串，使用像這樣子的語法(?no:text:other text)，
我們可以這樣子理解，如果代表 group $no 變數有符合的話，那麼就插入 text 否則就插入 other text ，
而 "other text" 這個選項是可省略，可將結果視為插入 "" ，這個功能非常強大，你將可以插入你所
想要的文字在你的程式片段中。


### 4.7.2 實際操作

轉換功能是非常強大，但是語法也相當的迂迴。
希望這些實際操作可以幫助你設計你所想要的轉換功能。


Demo: 將第一個字元轉成大寫
```
------------------- SNIP -------------------
snippet title "Title transformation"
${1:a text}
${1/\w+\s*/\u$0/}
endsnippet
------------------- SNAP -------------------
title<tab>big small ->
big small
Big small
```

Demo: 使用全域選項，將所有相符的結果第一個字元轉成大寫

```
------------------- SNIP -------------------
snippet title "Titlelize in the Transformation"
${1:a text}
${1/\w+\s*/\u$0/g}
endsnippet
------------------- SNAP -------------------
title<tab>this is a title ->
this is a title
This Is A Title
```

Demo: ASCII 轉換

```
------------------- SNIP -------------------
snippet ascii "Replace non ascii chars"
${1: an accentued text}
${1/.*/$0/a}
endsnippet
------------------- SNAP -------------------
ascii<tab>à la pêche aux moules
à la pêche aux moules
a la peche aux moules
```

Demo: 正規表示式的群組功能
      這是一個 C 語言中的 printf 的程式片段，第二個停頓點只有在第一個停頓點輸入有出現特定格式 （包含 % ）時才會出現。

```
------------------- SNIP -------------------
snippet printf
printf("${1:%s}\n"${1/([^%]|%%)*(%.)?.*/(?2:, :\);)/}$2${1/([^%]|%%)*(%.)?.*/(?2:\);)/}
endsnippet
------------------- SNAP -------------------
printf<tab>Hello<c-j> // End of line ->
printf("Hello\n"); // End of line

But
printf<tab>A is: %s<c-j>A<c-j> // End of line ->
printf("A is: %s\n", A); // End of line
```

其他還有更多可達成的轉換語法在 bundled snippets 中。
