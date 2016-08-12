---
layout: post
title: "Unicode escapes in Java source code are a bad idea"
excerpt:
modified: 2016-08-11 09:33:29 +0300
categories: articles
tags: [java, unicode]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---
Without saying anything else, I'm going to start this post with a simple code example.

{% highlight java %}
public class HelloUnicode {
    public static void main(String[] args) {
        // \u0055 is a Unicode escape for the capital U character (U)
        System.out.println("Hello \u0055nicode".length());
    }
}
{% endhighlight %}

Take a moment to think about what will be printed out? If you want, copy and paste the code to a new file, compile and run it. At first you might think `18` will be printed out since there's 18 characters between the double quotes.  But after reading the comment it is reasonable to think that the Unicode escape will be replaced with a single character and you're correct doing so. If you run the code, you will see that the program prints out `13`.

## What are *unicode escapes*

Unicode escapes are used to represent non-ASCII characters with ASCII characters. This will come in handy when you need to insert a character that cannot be represented in the source file's character set. According to [section 3.3 of the Java Language Specification (JLS)](https://docs.oracle.com/javase/specs/jls/se7/html/jls-3.html#jls-3.3 "Section 3.3 fo JLS") a *unicode escape* consists of a backslash character ('\') followed by one or more 'u' characters and four hexadecimal digits.

{% highlight text %}
UnicodeEscape:
    \ UnicodeMarker HexDigit HexDigit HexDigit HexDigit

UnicodeMarker:
    u
    UnicodeMarker u
{% endhighlight %}

So for example `\u000A` will be treated as a line feed. You can view a list of Unicode characters from [here](http://unicode-table.com/en/).
