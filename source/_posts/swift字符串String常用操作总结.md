---
title: swift字符串String常用操作总结
date: 2016-08-15 16:16:00
tags: [swift]
categories: swift
---

swift中字符串的操作

* 字符串的定义
```
var str1="hello, mandarava." //字符串变量
let str2="hello, mandarava." //字符串常量
let str3="" //空字符串
let str4=String() //空字符串

```

<!-- more -->

* 字符的定义
```
var char1:Character="m" //字符变量
let char2:Character="m" //字符常量

```

* 字符串的连接
```
let str1="hello, "
let str2="mandarava."
let str3=str1+str2 //=连接str1、str2
//str3="hello, mandarava."
//----------------------------------------
let str4="\(str1)\(str2)" //=连接str1、str2
//str4="hello, mandarava."
//----------------------------------------
let strArray=["apple", "orange", "cherry"]
let joinStr=",".join(strArray)
//joinStr="apple,orange,cherry"

```

* 字符串与字符的连接
```
let char1:Character="o"
var str1="hell"
let str2=str1+String(char1)
//str2="hello"
//----------------------------------------
let str3="\(str1)\(char1)"
//str3="hello"
//----------------------------------------
str1.append(char1)
//str1="hello"

```

* 字符串与其它类型值的连接
```
let xx=10
let yy=10.8
let str1="\(xx)+\(yy)=?"
//str1="10+10.8=?"
//----------------------------------------
let str2=String(format: "%i+%.1f=?", xx, yy)
//str2="10+10.8=?"

```

* 字符串枚举字符
```
//Swift 1.2
let str="mandarava"
for ch in str{
 println(ch)
}
//-----------------------
//Swift 2.0
let str="mandarava"
for ch in str.characters{
 print(ch)
}

```

* 获取字符串中指定索引处的字符
```
let str="Mandarava"
let chr=str[advance(str.startIndex,2)] //Swift 1.2 //chr:Character="n"
let chr=str[str.startIndex.advancedBy(2)] //Swift 2.0 //chr:Character="n"

```
* 计算字符串长度length
```
let str="@你好啊"
let len1=count(str) //swift 1.2 //=4
let len2=str.characters.count //swift 2.0 //=4
let blen=str.lengthOfBytesUsingEncoding(NSUTF8StringEncoding) //=10

```
* 字符串比较
```
let str1="hello,"
let str2="mandarava."
let str3="hello,mandarava."
let comp1 = str1==str2 //false
let comp2 = str1+str2 == str3 //true
let comp3 = str1 < str2 //true
let comp4 = str1 != str2 //true

```
* 是否包含子串contains
```
let str1="hello,mandarava."
let str2="mandarava"
let range=str1.rangeOfString(str2)
if range != nil{
 //包含
}

```
* 查找子串indexof
```
var str="hi,Mandarava."
let range=str.rangeOfString("Mandarava", options: NSStringCompareOptions.allZeros) //Swift 1.2
let range=str.rangeOfString("Mandarava", options: NSStringCompareOptions()) //Swift 2.0
let startIndex=range?.startIndex //=3

```
* 首字母大写capitalized
```
var str1="mandarava is a flower."
str1.capitalizedString
//str1="Mandarava Is A Flower.

```
* 转换为大写字母uppercase
```
var str1="hello, mandarava."
str1=str1.uppercaseString
//str1="HELLO, MANDARAVA."

```
* 转换为小写字母lowercase
```
var str1="HELLO, MANDARAVA."
str1=str1.lowercaseString
//str1="hello, mandarava."

```
* 截取字符串substring
```
let str1="hello,mandarava."
let str2=str1.substringFromIndex(advance(str1.startIndex, 6)) //Swift 1.2
let str2=str1.substringFromIndex(str1.startIndex.advancedBy(6)) //Swift 2.0
//str2="mandarava."
//----------------------------------------
let str3=str1.substringToIndex(advance(str1.startIndex, 5)) //Swift 1.2
let str3=str1.substringToIndex(str1.startIndex.advancedBy(5)) //Swift 2.0
//str3="hello"
//----------------------------------------
let range=Range<String.Index>(start: advance(str1.startIndex, 6), end: advance(str1.endIndex, -1)) //Swift 1.2
let range=Range<String.Index>(start: str1.startIndex.advancedBy(6), end: str1.endIndex.advancedBy(-1)) //Swift 2.0
let str4=str1.substringWithRange(range)
//str4="mandarava"

```
* 字符串修剪trim
```
let str1=" mandarava.\n "
let str2=str1.stringByTrimmingCharactersInSet(NSCharacterSet.whitespaceAndNewlineCharacterSet())
//str2="mandarava."
//----------------------------------------
let str3=str1.stringByTrimmingCharactersInSet(NSCharacterSet.whitespaceCharacterSet())
//str3="mandarava.\n"
//----------------------------------------
let charset=NSCharacterSet(charactersInString:" \n")
let str4=str1.stringByTrimmingCharactersInSet(charset)
//str4="mandarava."

```
* 字符串的分解子串split
```
var str1="boy, girl, man, woman"
let str1Array=str1.componentsSeparatedByString(",")
//str1Array=["boy", " girl", " man", " woman"]
var str2="boy,girl,man 10 20 30"
let charset=NSCharacterSet(charactersInString:", ")
let str2Array=str2.componentsSeparatedByCharactersInSet(charset)
//str2Array=["boy", "girl", "man", "10", "20", "30"]

```
* 字符串替换replace
```
var str1="My name is Mandarava."
let subRange=Range(start: str1.startIndex, end: advance(str1.startIndex, 2)) //Swift 1.2
let subRange=Range(start: str1.startIndex, end: str1.startIndex.advancedBy(2)) //Swift 2.0
str1.replaceRange(subRange, with: "Your")
//str1="Your name is Mandarava."
var str2="hello, Mandarava."
str2=str2.stringByReplacingOccurrencesOfString("Mandarava", withString: "你好啊")
//str2="hello, 你好啊."
str2=str2.stringByReplacingOccurrencesOfString("你好啊", withString: "Mandarava", options: NSStringCompareOptions.CaseInsensitiveSearch, range: nil)
//str2="hello, Mandarava."
string转换为Int/Long/Float/Double/Bool等
var str1="100"
var i=str1.toInt()! //Swift 1.2 //=100
var i=(str1 as NSString).integerValue //Swift 2.0 //=100
var i=(str1 as NSString).intValue //=100
var l=(str1 as NSString).longLongValue //=100
var str2="10.8"
var f=(str2 as NSString).floatValue //=10.8
var d=(str2 as NSString).doubleValue //=10.8
var str3="true"
var b=(str3 as NSString).boolValue //=true
```
