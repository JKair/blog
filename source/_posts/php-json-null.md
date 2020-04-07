---
title: PHP不标准的json
date: 2015.06.26 16:26:27
tags: [php]
categories: 后端
---

在使用PHP解析json的时候，可能你会发生一个问题，就是解析之后一直是null
那么这时候可能的原因是你使用了file_get_contents（）这样的函数，得到的数据前面有三个看不到的字符（无BOM 也是没用的），所以只要将得到的字符串，substr($str,3)就行了。