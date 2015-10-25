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
