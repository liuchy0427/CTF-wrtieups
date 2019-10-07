KeyChecker
---

下圖為透過  IDA pro 解碼後的結果

![After IDA pro](https://github.com/liuchy0427/CTF-wrtieups/Reverse/NCTU-csie/KeyChecker/images/擷取.PNG)

可以知道下面這段程式應該會是重點

![](https://github.com/liuchy0427/CTF-wrtieups/Reverse/NCTU-csie/KeyChecker/images/1.PNG)

在試用過 x64dbg 繞過所有判斷 function 之後，可以了解真正的 Flag 是需要與真正的 password 做XOR得到的

但是其實若仔細觀察code可以發現事有蹊蹺

```  c++
   for ( j = 0; j <= 18; ++j )
    {
      Buffer[j] ^= 2 * (RealYear + 63) + *(v6 + 2) + 127;
      if ( Buffer[j] != RealPassword[j] )
      {
        puts("[!] oops... time machine g0t some trouble in the 0ld tim3... ");
        break;
      }
    }
    
```

在第一個for 就給判斷是否為真正的密碼，所以若我們輸入的是正確的
那代表我們所輸入的前  18 個字元，就是真正的密碼

```  c++
    for ( k = 0; k <= 18; ++k )
          Buffer[k] ^= RealFlag[k];
        printf("[+] a flag found by time machine at %i:\n\t%s\n", RealYear);
      }
```

而在下一個 for 又是對前18個字元，也就是真正的密碼進行處理
所以可以得出結論:
**只要找到真正的密碼還有要與密碼做XOR的19個字，就可以得出FLAG!!!**

![](https://github.com/liuchy0427/CTF-wrtieups/Reverse/NCTU-csie/KeyChecker/images/2.PNG)

碰巧在 assemble 內就可以找到，我們需要的兩組數字
這時候寫一個簡單的 python 就可以得出我們要的 FLAG 了

``` python
  a = ['1D', '13', '10', '18', '51', '4C', '4F', '1C', '12', '51', '0B','8', '50', '51', '50', '51', '50', '51', '50']
  b = ['5B', '5F', '51', '5F', '2A', '1C', '0A', '43', '33', '2', '54','4D', '11', '2', '9', '2C', '70', '71', '70']

  for i in range(0,19):
    a[i] = int("0x"+a[i],16)
    b[i] = int("0x"+b[i],16)
    b[i] ^= a[i]
    print (chr(b[i]))
  print (a)
  print (b) 
```

得到



  
```
    FLAG{PE_!S_EASY}
```

