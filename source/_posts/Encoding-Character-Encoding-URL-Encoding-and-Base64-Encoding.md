---
title: 'Encoding: Character Encoding, URL Encoding and Base64 Encoding'
date: 2023-09-07 23:11:43
categories: encoding
tags: encoding
---

# **Character Encoding**

Computers only can understand binary. If we want to process text, we must first convert the text to binary before computers can process it.

## **ASCII**

In the early days of the internet, we didn’t need to worry about any characters other than english. We use [American Standard Code for Information Interchange (ASCII)](https://en.wikipedia.org/wiki/ASCII) , one byte (eight bits), to map a-z, A-Z , 0–9 and some control characters.

As you can imagine, there are hundreds of languages in the world. one byte is not enough for some language like Chinese or Japanese. Each country has its own standards, result in garbled characters displayed in multi-language mixed text.
<!--more-->

## **Unicode**

Unicode is sometimes called the [Universal Coded Character Set](https://en.wikipedia.org/wiki/Universal_Coded_Character_Set) (UCS), or even ISO/IEC 10646. Unicode is made up of lots of code points (mapping lots of characters from around the world to a key that all computers can reference.) A collection of code points is called a character set, which is what Unicode is. Unicode unifies all languages into one set of codes, so that there will be no more garbled characters.

## **Unicode Transform Protocol (UTF)**

UTF is a way we encode Unicode code points. The UTF encodings are defined by the Unicode standard, and are able to encode every single Unicode code point we need.

There are different types of UTF standards. They differ depending on the amount of bytes used to encode one code point. It also depends on whether you’re using UTF-8 (one byte per code point), UTF-16 (two bytes per code point) or UTF-32 (four bytes per code point).

To signal which Unicode character encoding (UTF-8, UTF-16, UTF-32) is used, [Byte order mark (BOM)](https://en.wikipedia.org/wiki/Byte_order_mark) is a two-byte marker at the beginning of a text that indicate what encoding is used.

## **UTF-8**

UTF-8 encodes all the Unicode code points from 0–127 in 1 byte (the same as ASCII). UTF encoding that converts Unicode encoding into “variable length character encoding” depend on the character you want to display. English letters are encoded into 1 byte ( 8 bits, the minimum number of bits that a UTF-8 code point will be) . Chinese and Japanese characters are are represented with 3 bytes. Some very rare characters will be encoded into 4–6 bytes. UTF-8 encoding can save space when transmitting and saving.

```

  character   |     ASCII     |    Unicode(UCS-16)  |      UTF-8
--------------|---------------|---------------------|-------------------
      A       |    01000001   |   00000000 01000001 |     01000001
```

## **HTML charset Attribute**

To display an HTML page correctly, a web browser must know the character set used in the page. This is specified in the `<meta>` tag:

```
<head>
  <meta charset="utf-8">
</head>
```

## **Text Storage**

Unicode encoding is uniformly used in computer memory. When we need to store text on disk or transmitted over internet, text is converted to UTF-8 encoding (may be other encoding, depend on your Operating System or encoding method) for space saving. When open file , the encoding text are converted to Unicode characters in memory.

> How can I see which encoding is used in a file
> 
> 
> *You can not really automatically find out whether a file was written with encoding X originally.*
> 
> *What you can easily do though is to verify whether the complete file can be successfully decoded somehow (but not necessarily correctly) using a specific codec. If you find any bytes that are not valid for a given encoding, it must be something else.*
> 
> *The problem is that many codecs are similar and have the same “valid byte patterns”, just interpreting them as different characters.*
> 

# **URL Encoding**

A URL (Uniform Resource Locator) is the address of a resource in the world wide web. URLs have a well-defined structure which was formulated in [RFC 1738](https://tools.ietf.org/html/rfc1738) . A URL is composed from characters including digits (0–9), letters(A-Z, a-z), and a few special characters ( “-”, “.”, “_”, “~”). If we want to transmit any character which is not a member of the above mentioned (unsafe characters like space, \, <, >, {, } , reserved characters like ?, /, #, :), we need to encode them so that web browsers can accept and understand. URL encoding is also called percent encoding since it uses percent sign (`%`) as an escape character.

```
http://www.example.com?name=@Test&phone=88888888

          encode value part in params       |
                                           \|?

http://www.example.com?name=%40Test&phone=%2B919999999999
```

# **Base64**

Base64 is a group of similar binary-to-text encoding schemes that represent binary data in an ASCII string format by translating it into a radix-64 representation. Base64 encoding schemes are commonly used when there is a need to encode binary data that needs to be stored and transferred over media that are designed to deal with ASCII. This is to ensure that the data remain intact without modification during transport. The resultant string of base 64 encoded data is always larger than the original. Another important distinction is that base 64 does not encrypt any information.

# **Reference**

- [Unicode Characters — What Every Developer Must Know About Encoding](https://www.freecodecamp.org/news/everything-you-need-to-know-about-encoding/)
- [分享幾個在 Windows 與 Linux 常見的編碼問題與解決方案](https://blog.miniasp.com/post/2021/08/05/Character-Encoding-Problems-for-Windows-and-Linux)
- [Why do you need to encode URLs?](https://stackoverflow.com/questions/2152738/why-do-you-need-to-encode-urls)
- [What is URL Encoding and How does it work?](https://www.urlencoder.io/learn/)
- [Why do we use Base64?](https://stackoverflow.com/questions/3538021/why-do-we-use-base64)
- [What is Base 64 Encoding?](https://bunny.net/academy/http/what-is-base64-encoding-and-decoding/)
- [Base64](https://developer.mozilla.org/en-US/docs/Glossary/Base64)