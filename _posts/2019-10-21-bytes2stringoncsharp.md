---  
layout: post  
title:  "C#中 byte/byte[] 转成16进制字符串"  
date:   2019-10-21 12:57:21  
categories: C#  
tags: C# byte  
---  

* content  
{:toc}  

### byte转16进制字符串string  

``` cs 
public static string ByteToHex(byte b)  
{  
    string cs = Convert.ToString(b, 16);  
    if (cs.Length == 1)  
    {  
        cs = "0" + cs;  
    }  
    return cs;  
}  
```  

### byte[]转16进制字符串string  

``` cs 
public static string BytesToHex(byte[] b, int length)  
{  
    string s = "";  
    for (int i = 0; i < length; i++)  
    {  
        string cs = Convert.ToString(b[i], 16);  
        if (cs.Length == 1)  
        {  
            cs = "0" + cs;  
        }  
        s += (cs + " ");  
    }  
    return s;  
}  
```  