

PHP 的 [str\_replace()](https://www.php.net/manual/en/function.str-replace.php)、[str\_ireplace()](https://www.php.net/manual/en/function.str-ireplace.php) 在使用上，剛開始覺得有些不太直覺的地方，不過，後來發現應該是文件沒有看仔細的因素~

-   註：str\_ireplace() 是不管大小寫，用法都一樣，此篇範例都用 str\_ireplace() 來做。

PHP 如果想要將一個字串的英文字字首轉大寫，可以使用 [ucwords()](https://www.php.net/manual/zh/function.ucwords.php)，但是會遇到下面產品大寫的位置不同，所以使用 str\_replace() 怎麼寫法會比較容易懂呢？

想要將字串做 1 對 1 的 Mapping 轉換，最好的方式是一個一對一的陣列，例如：

```
$replace_mapping = [
    'iphone'  => 'iPhone',
    'ipad'    => 'iPad',
    'imac'    => 'iMac',
    'airpods' => 'AirPods',
];
```

但是 str\_ireplace($find, $replace, $source) 是需要塞兩個不同的陣列，例如：

1.  $search = \['iphone', 'ipad', 'imac', 'airpods'\];
2.  $replace = \['iPhone', 'iPad', 'iMac', 'AirPods'\];
3.  str\_ireplace($search, $replace, $source)

所以想要用剛剛原始的 $replace\_mapping 來做，會寫成下述：

1.  $search = array\_keys($eplace\_mapping);
2.  $replace = array\_values($eplace\_mapping);
3.  str\_ireplace($search, $replace, $source)

這種寫法是可以用，但是總覺得用 values 這個比較慢，而且做法有點蠢~

再仔細看文件說明，發現說明是這樣說的：

-   search
    -   The value being searched for, otherwise known as the needle. An array may be used to designate multiple needles.
-   replace
    -   The replacement value that replaces found **search values**. An array may be used to designate multiple replacements.
-   註：replace 那個會取用 search 的 values

於是一樣 Array mapping，但是完整寫法如下：

1.  <?php
    
2.  $replace\_mapping \= \[
    
3.  'iphone' \=\> 'iPhone',
    
4.  'ipad' \=\> 'iPad',
    
5.  'imac' \=\> 'iMac',
    
6.  'airpods' \=\> 'AirPods',
    
7.  \];
    
8.  echo [str\_ireplace](http://www.php.net/manual-lookup.php?pattern=str_ireplace)([array\_keys](http://www.php.net/manual-lookup.php?pattern=array_keys)($replace\_mapping), $replace\_mapping, 'anbc iphone samsung ipad asamsungaaa IPHONE aa');
    

10.  ?>
    

## 文章導覽