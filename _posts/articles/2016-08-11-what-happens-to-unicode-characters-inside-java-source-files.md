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

Now equipped with the knowledge that Unicode escapes are replaced with their respective Unicode character, let's look at the following example.

{% highlight java %}
public class NewLine {
    public static void main(String[] args) {
        // \u000A is a unicode escape for the line feed (LF)
        // \u0055 is a Unicode escape for the capital U character (U)
        System.out.println("Hello \u0055nicode".length());
    }
}
{% endhighlight %}

Can you guess what will be printed out now? The answer should be the same as before, right?. I'm sure some of you might suspect that this is a trick question and as a matter of fact, it is. The previous example will not compile at all.

{% highlight bash %}
$ javac NewLine.java
NewLine.java:5: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                      ^
NewLine.java:5: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                                     ^
NewLine.java:5: error: '(' expected
        // \u000A is a unicode escape for the line feed (LF)
                                         ^
NewLine.java:5: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                                                  ^
NewLine.java:5: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                                                            ^
NewLine.java:7: error: ')' expected
        System.out.println("Hello \u0055nicode".length());
                                                         ^
6 errors
{% endhighlight %}

What!? So many errors! My IDE doesn't show any squiggly red lines and I can't seem to find any syntax errors myself. Error on line 5? But that's a comment. What is going on?

![Screenshot of my IDE][ide]

In this post I'm going to cover Unicode escapes in Java and by the end of this post you'll understand what caused the error in the first place.

## What are *unicode escapes*

Let's start from the beginning. Unicode escapes are used to represent any Unicode characters with only ASCII characters. This will come in handy when you need to insert a character that cannot be represented in the source file's character set. According to [section 3.3 of the Java Language Specification (JLS)](https://docs.oracle.com/javase/specs/jls/se7/html/jls-3.html#jls-3.3 "Section 3.3 fo JLS") a *unicode escape* consists of a backslash character ('\') followed by one or more 'u' characters and four hexadecimal digits.

{% highlight text %}
UnicodeEscape:
    \ UnicodeMarker HexDigit HexDigit HexDigit HexDigit

UnicodeMarker:
    u
    UnicodeMarker u
{% endhighlight %}

So for example `\u000A` will be treated as a line feed. Looking back at the second code example in this post, a Unicode escape representing the line feed was used inside a comment. You can view a list of Unicode characters from [here](http://unicode-table.com/en/).

## What causes it?

To get a better understanding of what is going on, we need to look at [section 3.2 of the Java Language Specification - Lexical Translations](https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.2 "Section 3.2 Lexical Translations"). I cannot speak for all compilers that have ever existed but usually the first job of the compiler is to take the source code of the program, treat it as a sequence of characters and produce a sequence of tokens. A token is something that has a meaning in the context of the language. For example it can be a [reserved word](https://en.wikipedia.org/wiki/Reserved_word "Wikipedia page for reserved words") (`public`, `class` or `interface`), an [operator](https://en.wikipedia.org/wiki/Operator_(computer_programming) "Wikipedia page for operators") (`+`, `>>`) or a [literal](https://en.wikipedia.org/wiki/Literal_(computer_programming) "Wikipedia page for literals") (a notation for representing a fixed value). The process of generating tokens from a sequence of characters is called [lexical analysis](https://en.wikipedia.org/wiki/Lexical_analysis "Wikipedia page for lexical analysis") (or lexical translation as it is called in the Oracle docs) and the program that performs that is called a *lexer* or a *tokenizer*.

The [Java Language Specification]((https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.2 "Section 3.2 Lexical Translations")) says that lexical translation is performed in the following 3 steps, where each step is applied to the result of the previous step:

1. Translation of Unicode escapes.
2. Divide stream of input characters into lines by recognizing line terminators (LF, CR or CR LF).
3. Discard whitespace and comments and tokenize the result from the previous step.

As you can see, the very first step processes Unicode escapes. This is done before the compiler has had the change to separate the source code into tokens. Broadly speaking, this is like applying a *search and replace* function on the source code, replacing all **well-formed** Unicode escapes with their respective Unicode characters, and then letting the compiler work on the program.

Keep in mind, when Unicode escapes are being processed, the compiler does not differentiate comments and actual code. It can only see a sequence of characters. And this explains the erroneous code you saw in the introduction of this post. Let's have a look at it again.

{% highlight java %}
//This is the original source code
public class NewLine {
    public static void main(String[] args) {
        // \u000A is a unicode escape for the line feed (LF)
        // \u0055 is a Unicode escape for the capital U character (U)
        System.out.println("Hello \u0055nicode".length());
    }
}

//This is what it looks like after Unicode escapes have been processed
public class NewLine {
    public static void main(String[] args) {
        //
 is a unicode escape for the line feed (LF)
        // U is a Unicode escape for the capital U character (U)
        System.out.println("Hello Unicode".length());
    }
}
{% endhighlight %}

The Unicode escape representing the line feed character is replaced with a line feed and now part of the comment is on a new line. Unfortunately the new line does not start with a *double-slash* (`//`) and the rest of the line is not valid Java code. Hence the confusing compiler error shown previously.

You can try it yourself. Java is bundled with a tool called *[native2ascii](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/native2ascii.html "Oracle docs about native2ascii")* which converts a file with characters in any supported character encoding to one with ASCII and/or Unicode escapes, or visa versa.

{% highlight bash %}
$ native2ascii -reverse NewLine.java

public class NewLine {
    public static void main(String[] args) {
        //
 is a unicode escape for the line feed (LF)
        // U is a Unicode escape for the capital U character (U)
        System.out.println("Hello Unicode".length());
    }
}

{% endhighlight %}

## Hiding code in comments

## why is it designed like that
## what should i use instead
## windows path example
## whole program with unicode escapes

[ide]: {{ site.url }}/images/2016-08-11-unicode-escapes/ide.png "Screenshot of my IDE"
