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

Take a moment to think about what will be printed out? If you want, copy and paste the code to a new file, compile and run it. At first glance it looks like `18` will be printed out since there's 18 characters between the double quotes.  But after reading the comment it is reasonable to think that the Unicode escape will be replaced with a single character. If you run the code, you will see that the program prints out `13` which proves the hypothesis that Unicode escape sequences are in fact removed and replaced with a single character inside a string literal.

Now equipped with the knowledge that Unicode escapes are replaced with their respective Unicode characters, let's look at the following example.

{% highlight java %}
public class NewLine {
    public static void main(String[] args) {
        // \u000A is a unicode escape for the line feed (LF)
        // \u0055 is a Unicode escape for the capital U character (U)
        System.out.println("Hello \u0055nicode".length());
    }
}
{% endhighlight %}

Can you guess what will be printed out now? The answer should be the same as before, right?. I'm sure some of you might suspect that this is a trick question and as a matter of fact, it is. This example will not compile at all.

{% highlight bash %}
$ javac NewLine.java
NewLine.java:3: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                      ^
NewLine.java:3: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                                     ^
NewLine.java:3: error: '(' expected
        // \u000A is a unicode escape for the line feed (LF)
                                         ^
NewLine.java:3: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                                                  ^
NewLine.java:3: error: ';' expected
        // \u000A is a unicode escape for the line feed (LF)
                                                            ^
NewLine.java:5: error: ')' expected
        System.out.println("Hello \u0055nicode".length());
                                                         ^
6 errors
{% endhighlight %}

What!? So many errors! My IDE doesn't show any squiggly red lines and I can't seem to find any syntax errors myself. Error on line 3? But that's a comment. What is going on?

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

## What caused the error?

To get a better understanding of what is going on, we need to look at [section 3.2 of the Java Language Specification - Lexical Translations](https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.2 "Section 3.2 Lexical Translations"). I cannot speak for all compilers that have ever existed but usually the first job of the compiler is to take the source code of the program, treat it as a sequence of characters and produce a sequence of tokens. A token is something that has a meaning in the context of the language. For example it can be a [reserved word](https://en.wikipedia.org/wiki/Reserved_word "Wikipedia page for reserved words") (`public`, `class` or `interface`), an [operator](https://en.wikipedia.org/wiki/Operator_(computer_programming) "Wikipedia page for operators") (`+`, `>>`) or a [literal](https://en.wikipedia.org/wiki/Literal_(computer_programming) "Wikipedia page for literals") (a notation for representing a fixed value). The process of generating tokens from a sequence of characters is called [lexical analysis](https://en.wikipedia.org/wiki/Lexical_analysis "Wikipedia page for lexical analysis") (or lexical translation as it is called in the Oracle docs) and the program that performs that is called a *lexer* or a *tokenizer*.

The [Java Language Specification]((https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.2 "Section 3.2 Lexical Translations")) says that lexical translation is performed in the following 3 steps, where each step is applied to the result of the previous step:

1. Translation of Unicode escapes.
2. Divide stream of input characters into lines by recognizing line terminators (LF, CR or CR LF).
3. Discard whitespace and comments and tokenize the result from the previous step.

As you can see, the very first step processes Unicode escapes. This is done before the compiler has had the change to separate the source code into tokens. Broadly speaking, this is like applying a *search and replace* function on the source code, replacing all **well formed** Unicode escapes with their respective Unicode characters, and then letting the compiler work on the rest of the code.

Keep in mind, when Unicode escapes are being processed, the compiler does not differentiate comments from actual code. It can only see a sequence of characters. And this explains the erroneous code you saw in the introduction of this post. Let's have a look at it again.

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

If Unicode escapes are processed before everything else, then can I cleverly hide code inside comments which will later be executed? The answer to this question is yes. Looking back at the previous example, we saw that a line feed was inserted and the rest of the comment was on the next line, resulting in invalid Java code. But we could have written the following

{% highlight java %}
public class HidingCode {
    public static void main(String[] args) {
        //\u000A System.out.println("This is a comment");
        System.out.println("Hello world");
    }
}

If we replace the Unicode escape with a line feed, then it should be clear that there's actually two print statements executed.

{% highlight bash %}
native2ascii -reverse HidingCode.java

public class HidingCode {
    public static void main(String[] args) {
        //
 System.out.println("This is a comment");
        System.out.println("Hello world");
    }
}
{% endhighlight %}


//prints out
This is a comment
Hello world
{% endhighlight %}

## Why does Java allow that?

This all seems weird, right? Why is Java designed like that? Is it a bug that was accidentally introduced and never fixed because it would break something else? To find an answer to that question we need to look back at [section 3.3 of the Java Language Specification (JLS)](https://docs.oracle.com/javase/specs/jls/se7/html/jls-3.html#jls-3.3 "Section 3.3 of JLS") and also [section 3.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.1 "Section 3.1 of JLS").

From section 3.1

> The Java programming language represents text in sequences of 16-bit code units, using the UTF-16 encoding.

From section 3.3

> The Java programming language specifies a standard way of transforming a program written in Unicode into ASCII that changes a program into a form that can be processed by ASCII-based tools. The transformation involves converting any Unicode escapes in the source text of the program to ASCII by adding an extra u - for example, \uxxxx becomes \uuxxxx - while simultaneously converting non-ASCII characters in the source text to Unicode escapes containing a single u each.

Unicode escapes were designed to ensure compatibility with a wide variety of character sets. Think of the following scenario. You receive a piece of code with an encoding your text editor does not understand (i.e. the code includes characters not available in the encoding you use). This can be solved by transforming all unknown characters with Unicode escapes. As ASCII is the lowest common denominator of character sets, it is always possible to represent Java code in any encoding by replacing characters that are not supported by the target encoding with Unicode escapes. Today Unicode is fairly common and this should not be an issue, but I guess back in the early days this was useful.

The transformed version is equal to the initial version and the compiler treats them as the same. As this process is reversible, the compiler can go back to the initial version by replacing Unicode escapes with respective Unicode characters.

From section 3.3

> This transformed version is equally acceptable to a Java compiler and represents the exact same program. The exact Unicode source can later be restored from this ASCII form by converting each escape sequence where multiple u's are present to a sequence of Unicode characters with one fewer u, while simultaneously converting each escape sequence with a single u to the corresponding single Unicode character.

## Prefer escape sequences

Because Unicode escapes are processed before everything else in the compilation process, they can create a considerable amount of confusion. Therefore it is better to avoid them if possible. Prefer [escape sequences](https://docs.oracle.com/javase/tutorial/java/data/characters.html "Java escape sequences") if you need to represent line feeds, double quotes and the like in string or character literals. There's no need to use Unicode escapes for ASCII characters.

## Unicode escapes have to be well formed

I mentioned previously that only **well formed** Unicode escapes are replaced with Unicode characters during the compilation process. You will get an error if there's an ill-formed Unicode escape in your code. Have a look at the following example.

{% highlight java %}
public class IllFormedUnicodeEscape {
    public static void main(String[] args) {
        // user data is read from C:\data\users\profile
        System.out.println("User data");
    }
}
{% endhighlight %}

This seems like an innocent looking piece of code. The comment tries to be helpful and communicate something important to the reader. Unfortunately there's an Unicode escape lurking in this code which is not well formed. Windows path names use backslashes as do Unicode escapes to denote the start of the escape sequence. In this example, the path name contains a folder named `users`. As you know by now, Unicode escapes start with `\u` and the compiler expects four hexadecimal digits to be followed. When this rule is not met, the compiler will throw an error.

## Taking it to the extreme

We have looked at several examples of Unicode escape misuse. Your eye should be trained enough to spot most of them by now. For the next example I'm going to show you a piece of code that I first saw when I read the book [Java Puzzlers](http://www.javapuzzlers.com/ "Java Puzzlers book") by Joshua Bloch and Neal Gafter.

{% highlight java %}
\u0070\u0075\u0062\u006c\u0069\u0063\u0020\u0020\u0020\u0020
\u0063\u006c\u0061\u0073\u0073\u0020\u0055\u0067\u006c\u0079
\u007b\u0070\u0075\u0062\u006c\u0069\u0063\u0020\u0020\u0020
\u0020\u0020\u0020\u0020\u0073\u0074\u0061\u0074\u0069\u0063
\u0076\u006f\u0069\u0064\u0020\u006d\u0061\u0069\u006e\u0028
\u0053\u0074\u0072\u0069\u006e\u0067\u005b\u005d\u0020\u0020
\u0020\u0020\u0020\u0020\u0061\u0072\u0067\u0073\u0029\u007b
\u0053\u0079\u0073\u0074\u0065\u006d\u002e\u006f\u0075\u0074
\u002e\u0070\u0072\u0069\u006e\u0074\u006c\u006e\u0028\u0020
\u0022\u0048\u0065\u006c\u006c\u006f\u0020\u0077\u0022\u002b
\u0022\u006f\u0072\u006c\u0064\u0022\u0029\u003b\u007d\u007d
{% endhighlight %}

What? Really? This looks like an entry to a [code obfuscation contest](http://www.ioccc.org/ "The International Obfuscated C Code Contest"). But if you think about it, this seems like it should compile, granted that all the Unicode escapes actually represent characters that make up a valid Java program. We learned that the very first thing the compiler does is look for Unicode escapes and replace them. It does not know anything about the program structure at that point.

You can try it yourself. Copy the text into a file called `Ugly.java`. Then compile and run the program. By the way, there's no point trying to run it from an IDE (at least IntelliJ IDEA is baffled and is only able to show squiggly red lines). Use command line tools instead.

{% highlight bash %}
$ javac Ugly.java
$ java Ugly
Hello world
{% endhighlight %}

Additionally you can use the `native2ascii` tool to view what the code looks like if all Unicode escapes have been replaced.

{% highlight bash %}
$ native2ascii -reverse Ugly.java
public
class Ugly
{public
    static
void main(
String[]  
    args){
System.out
.println(
"Hello w"+
"orld");}}
{% endhighlight %}

I have only one thing to say. Just because you can doesn't mean you should.

## Summary

Even if you did or did not know about how Unicode escapes behave in Java, I'm guessing this information is of little use to you in your day to day life. That's because most of the time there's no need to insert a Unicode escape. And even if you do, [most operating systems have a method to input them](https://en.wikipedia.org/wiki/Unicode_input#In_X11_.28Linux_and_other_Unix_variants.29 "Unicode input"). Unicode is fairly common nowadays and most IDE's and text editors can display non-ASCII characters. If possible, avoid Unicode escapes because they create confusion. Prefer escape sequences instead.



[ide]: {{ site.url }}/images/2016-08-11-unicode-escapes/ide.png "Screenshot of my IDE"
