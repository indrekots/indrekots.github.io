---
layout: post
title: "Improve the Quality of Your Tests With Mutation Testing"
excerpt: "Ever wondered if your tests are actually catching bugs as they should? Let's dive into mutation testing, a clever technique that helps us validate the quality of our tests."
modified: 2024-04-012 12:05:18 +0200
categories: articles
tags: [testing, java, pit]
image:
  path: /images/2024-04-12-improve-the-quality-of-your-tests-with-mutation-testing/cover.jpg
  thumbnail: /images/2024-04-12-improve-the-quality-of-your-tests-with-mutation-testing/cover_thumb.jpg
  caption: "Photo by [ANIRUDH](https://unsplash.com/photos/a-close-up-of-a-double-strand-of-gold-glitter-YQYacLW8o2U)"
comments: true
share: true
published: true
aging: false
amazon_links: false
---

As software engineers, we write tests to ensure the correctness of our code.
By rerunning tests after each change, we can quickly identify if any modifications have inadvertently introduced new bugs or broken existing functionality.
The existing set of tests form a safety net that allows us to move faster with less risk of breaking things.
But how can we validate the quality of the tests themselves?
This post will explore the concept of mutation testing, a potential answer to that very question.

## What is Mutation Testing?

Imagine a scenario where you edit existing functionality but accidentally introduce a bug.
This can happen to any of us.
All is well and good if your tests start to fail.
A failing test means that the functionality is sufficiently covered with tests.
Your safety net prevented a potential issue.
However, if no tests fail after you introduce the bug, it may indicate that certain parts of the code are not being properly tested or that the tests themselves are inadequate.

This is essentially the premise behind mutation testing.
**Instead of you introducing bugs accidentally, a mutation testing tool will do that deliberately.**
A mutation testing tool introduces a change of behavior to the code.
Consider, for example, negating or removing a conditional, or returning `null` from a method call.
The altered version of the code is referred to as a *mutant*.

Tests are executed after each mutation.
The mutant is *killed* if at least one test fails, indicating that the tests discovered a noticable change in behavior.
If no tests fail, the mutant *has survived*, suggesting there are gaps in our tests or we have tests that never fail.
However, in some cases, it can also mean that the code itself is meaningless and isn't actually needed.

We can draw a parallel to chaos engineering.

> Chaos Engineering is the discipline of experimenting on a system in order to build confidence in the system’s capability to withstand turbulent conditions in production.
> <footer><a href="https://principlesofchaos.org/">Principles of Chaos Engineering</a></footer>

Mutation testing, however, *experiments* on our code and it's goal is to introduce changes that make the tests fail.
This helps us to discover weaknesses in our test suite.

## Show me the Code

Let's have a look at an example scenario using [PIT](https://pitest.org/), a mutation testing tool for Java.
The following is the method under test.
It calculates income tax using dual rates.
All income up to 10,000 is charged with 20% and everything above 10,000 is charged with 25%.
Ignore the fact that we're dealing with money and we use `double`.
It's just an example.

```java
class IncomeTaxCalculator {

    private static final double INCOME_TAX_20 = 0.2;
    private static final double INCOME_TAX_25 = 0.25;
    private static final double INCOME_TAX_25_BRACKET = 10_000.00;

    public static double calculateIncomeTax(double income) {
        var incomeTax = 0.00;
        var remainingTaxableIncome = income;

        if (income > INCOME_TAX_25_BRACKET) {
            var incomeOver25PercentBracket = income - INCOME_TAX_25_BRACKET;
            incomeTax += incomeOver25PercentBracket * INCOME_TAX_25;
            remainingTaxableIncome -= incomeOver25PercentBracket;
        }

        return incomeTax + (remainingTaxableIncome * INCOME_TAX_20);
    }
}
```

Let's create our first test.

```java
@Test
void calculateTaxBracket20() {
    assertEquals(1600.00, IncomeTaxCalculator.calculateIncomeTax(8_000.00));
}
```

The following is the PIT mutation testing report.
It shows which mutations were applied and whether the mutants were killed or not.
It resembles a code coverage report.
We can clearly see that the if-branch of the code was never executed.

<style>
.src { 
    border: 1px solid #dddddd;
    border-collapse: separate;
    padding-top: 10px;
    padding-right: 5px;
    padding-left: 5px;
    font-family: Consolas, Courier, monospace;
}

.src p {
  margin-bottom: 0;
}

.src pre {
  margin: 0;
}

.src td {
  padding: 0;
}

.covered, .COVERED {
    background-color: #ddffdd;
}

.uncovered, .UNCOVERED {
    background-color: #ffdddd;
}

.killed, .KILLED {
    background-color: #aaffaa;
}

.survived, .SURVIVED {
    background-color: #ffaaaa;
}

.uncertain, .UNCERTAIN {
    background-color: #dde7ef;
}

.run_error, .RUN_ERROR {
    background-color: #dde7ef;
}

.na {
    background-color: #eeeeee;
}

.timed_out, .TIMED_OUT {
    background-color: #dde7ef;
}

.non_viable, .NON_VIABLE {
    background-color: #aaffaa;
}

.memory_error, .MEMORY_ERROR {
    background-color: #dde7ef;
}

.not_started, .NO_STARTED {
    background-color: #dde7ef; color : red
}

.no_coverage, .NO_COVERAGE {
    background-color: #ffaaaa;
}

.tests {
    width: 50%;
    float: left;
}

.mutees {
    float: right;
    width: 50%;
}

.unit {
    padding-top: 20px;
    clear: both;
}

.coverage_bar {
    display: inline-block;
    height: 1.1em;
    width: 130px;
    background: #FAA;
    margin: 0 5px;
    vertical-align: middle;
    border: 1px solid #AAA;
    position: relative;
}

.coverage_complete {
    display: inline-block;
    height: 100%;
    background: #DFD;
    float: left;
}

.coverage_legend {
    position: absolute;
    height: 100%;
    width: 100%;
    left: 0;
    top: 0;
    text-align: center;
}

.line, .mut {
    vertical-align: middle;
}

.coverage_percentage {
    display: inline-block;
    width: 3em;
    text-align: right;
}

.pop {
    outline:none;
}

.pop strong {
    line-height: 30px;
}

.pop {
    text-decoration: none;
}

.pop span {
    z-index: 10;
    display: none;
    padding: 14px 20px;
    margin-top: -30px;
    margin-left: 28px;
    width: 800px;
    line-height: 16px;
    word-wrap: break-word;
    border-radius: 4px;
    -moz-border-radius: 4px;
    -webkit-border-radius: 4px;
    -moz-box-shadow: 5px 5px 8px #CCC;
    -webkit-box-shadow: 5px 5px 8px #CCC;
    box-shadow: 5px 5px 8px #CCC;
}

.pop:hover span {
    display: inline;
    position: absolute;
    color: #111;
    border: 1px solid #DCA;
    background: #fffAF0;
}

.width-1 {
    width: 1%;
}

.width-2 {
    width: 2%;
}

.width-3 {
    width: 3%;
}

.width-4 {
    width: 4%;
}

.width-5 {
    width: 5%;
}

.width-6 {
    width: 6%;
}

.width-7 {
    width: 7%;
}

.width-8 {
    width: 8%;
}

.width-9 {
    width: 9%;
}

.width-10 {
    width: 10%;
}

.width-11 {
    width: 11%;
}

.width-12 {
    width: 12%;
}

.width-13 {
    width: 13%;
}

.width-14 {
    width: 14%;
}

.width-15 {
    width: 15%;
}

.width-16 {
    width: 16%;
}

.width-17 {
    width: 17%;
}

.width-18 {
    width: 18%;
}

.width-19 {
    width: 19%;
}

.width-20 {
    width: 20%;
}

.width-21 {
    width: 21%;
}

.width-22 {
    width: 22%;
}

.width-23 {
    width: 23%;
}

.width-24 {
    width: 24%;
}

.width-25 {
    width: 25%;
}

.width-26 {
    width: 26%;
}

.width-27 {
    width: 27%;
}

.width-28 {
    width: 28%;
}

.width-29 {
    width: 29%;
}

.width-30 {
    width: 30%;
}

.width-31 {
    width: 31%;
}

.width-32 {
    width: 32%;
}

.width-33 {
    width: 33%;
}

.width-34 {
    width: 34%;
}

.width-35 {
    width: 35%;
}

.width-36 {
    width: 36%;
}

.width-37 {
    width: 37%;
}

.width-38 {
    width: 38%;
}

.width-39 {
    width: 39%;
}

.width-40 {
    width: 40%;
}

.width-41 {
    width: 41%;
}

.width-42 {
    width: 42%;
}

.width-43 {
    width: 43%;
}

.width-44 {
    width: 44%;
}

.width-45 {
    width: 45%;
}

.width-46 {
    width: 46%;
}

.width-47 {
    width: 47%;
}

.width-48 {
    width: 48%;
}

.width-49 {
    width: 49%;
}

.width-50 {
    width: 50%;
}

.width-51 {
    width: 51%;
}

.width-52 {
    width: 52%;
}

.width-53 {
    width: 53%;
}

.width-54 {
    width: 54%;
}

.width-55 {
    width: 55%;
}

.width-56 {
    width: 56%;
}

.width-57 {
    width: 57%;
}

.width-58 {
    width: 58%;
}

.width-59 {
    width: 59%;
}

.width-60 {
    width: 60%;
}

.width-61 {
    width: 61%;
}

.width-62 {
    width: 62%;
}

.width-63 {
    width: 63%;
}

.width-64 {
    width: 64%;
}

.width-65 {
    width: 65%;
}

.width-66 {
    width: 66%;
}

.width-67 {
    width: 67%;
}

.width-68 {
    width: 68%;
}

.width-69 {
    width: 69%;
}

.width-70 {
    width: 70%;
}

.width-71 {
    width: 71%;
}

.width-72 {
    width: 72%;
}

.width-73 {
    width: 73%;
}

.width-74 {
    width: 74%;
}

.width-75 {
    width: 75%;
}

.width-76 {
    width: 76%;
}

.width-77 {
    width: 77%;
}

.width-78 {
    width: 78%;
}

.width-79 {
    width: 79%;
}

.width-80 {
    width: 80%;
}

.width-81 {
    width: 81%;
}

.width-82 {
    width: 82%;
}

.width-83 {
    width: 83%;
}

.width-84 {
    width: 84%;
}

.width-85 {
    width: 85%;
}

.width-86 {
    width: 86%;
}

.width-87 {
    width: 87%;
}

.width-88 {
    width: 88%;
}

.width-89 {
    width: 89%;
}

.width-90 {
    width: 90%;
}

.width-91 {
    width: 91%;
}

.width-92 {
    width: 92%;
}

.width-93 {
    width: 93%;
}

.width-94 {
    width: 94%;
}

.width-95 {
    width: 95%;
}

.width-96 {
    width: 96%;
}

.width-97 {
    width: 97%;
}

.width-98 {
    width: 98%;
}

.width-99 {
    width: 99%;
}

.width-100 {
    width: 100%;
}
</style>

<table class="src">



<tbody><tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_1">
1
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_1"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">package test.pit;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_2">
2
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_2"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="uncovered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_3">
3
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_3"></a>
<span>
</span>
</span>
</td>
<td class="uncovered"><pre><span class="">class IncomeTaxCalculator {</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_4">
4
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_4"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_5">
5
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_5"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    private static final double INCOME_TAX_20 = 0.2;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_6">
6
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_6"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    private static final double INCOME_TAX_25 = 0.25;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_7">
7
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_7"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    private static final double INCOME_TAX_25_BRACKET = 10_000.00;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_8">
8
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_8"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_9">
9
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_9"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    public static double calculateIncomeTax(double income) {</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_10">
10
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_10"></a>
<span>
</span>
</span>
</td>
<td class="covered"><pre><span class="">        var incomeTax = 0.00;</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_11">
11
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_11"></a>
<span>
</span>
</span>
</td>
<td class="covered"><pre><span class="">        var remainingTaxableIncome = income;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_12">
12
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_12"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_13">
13
</a></td>
<td class="survived">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_13">2</a>
<span>
1. calculateIncomeTax : changed conditional boundary → SURVIVED<br>
2. calculateIncomeTax : negated conditional → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="survived">        if (income &gt; INCOME_TAX_25_BRACKET) {</span></pre></td></tr>


<tr>
<td class="uncovered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_14">
14
</a></td>
<td class="survived">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_14">1</a>
<span>
1. calculateIncomeTax : Replaced double subtraction with addition → NO_COVERAGE<br>

</span>
</span>
</td>
<td class="uncovered"><pre><span class="survived">            var incomeOver25PercentBracket = income - INCOME_TAX_25_BRACKET;</span></pre></td></tr>


<tr>
<td class="uncovered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_15">
15
</a></td>
<td class="survived">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_15">2</a>
<span>
1. calculateIncomeTax : Replaced double multiplication with division → NO_COVERAGE<br>
2. calculateIncomeTax : Replaced double addition with subtraction → NO_COVERAGE<br>

</span>
</span>
</td>
<td class="uncovered"><pre><span class="survived">            incomeTax += incomeOver25PercentBracket * INCOME_TAX_25;</span></pre></td></tr>


<tr>
<td class="uncovered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_16">
16
</a></td>
<td class="survived">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_16">1</a>
<span>
1. calculateIncomeTax : Replaced double subtraction with addition → NO_COVERAGE<br>

</span>
</span>
</td>
<td class="uncovered"><pre><span class="survived">            remainingTaxableIncome -= incomeOver25PercentBracket;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_17">
17
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_17"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">        }</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_18">
18
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_18"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_19">
19
</a></td>
<td class="killed">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_19">3</a>
<span>
1. calculateIncomeTax : replaced double return with 0.0d for test/pit/IncomeTaxCalculator::calculateIncomeTax → KILLED<br>
2. calculateIncomeTax : Replaced double addition with subtraction → KILLED<br>
3. calculateIncomeTax : Replaced double multiplication with division → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="killed">        return incomeTax + (remainingTaxableIncome * INCOME_TAX_20);</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_20">
20
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_20"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    }</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@510da778_21">
21
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@510da778_21"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">}</span></pre></td></tr>



<tr><td></td><td></td><td><h2>Mutations</h2></td></tr>

<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@510da778_13">13</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@510da778_13"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket20()]</span></span> negated conditional → KILLED</p> <p class="SURVIVED"><span class="pop">2.<span><b>2</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>none</span></span> changed conditional boundary → SURVIVED</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@510da778_14">14</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@510da778_14"> 

<p class="NO_COVERAGE"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>none</span></span> Replaced double subtraction with addition → NO_COVERAGE</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@510da778_15">15</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@510da778_15"> 

<p class="NO_COVERAGE"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>none</span></span> Replaced double multiplication with division → NO_COVERAGE</p> <p class="NO_COVERAGE"><span class="pop">2.<span><b>2</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>none</span></span> Replaced double addition with subtraction → NO_COVERAGE</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@510da778_16">16</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@510da778_16"> 

<p class="NO_COVERAGE"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>none</span></span> Replaced double subtraction with addition → NO_COVERAGE</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@510da778_19">19</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@510da778_19"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket20()]</span></span> replaced double return with 0.0d for test/pit/IncomeTaxCalculator::calculateIncomeTax → KILLED</p> <p class="KILLED"><span class="pop">2.<span><b>2</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket20()]</span></span> Replaced double addition with subtraction → KILLED</p> <p class="KILLED"><span class="pop">3.<span><b>3</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket20()]</span></span> Replaced double multiplication with division → KILLED</p> 
</a></td>
</tr>

</tbody></table>

To kill the mutants from the if-branch, we need to add couple more tests.
Let's add a test where the income is greater than 10,000 and let's also test the boundary condition.

```java
@Test
void calculateTaxBracket25() {
    assertEquals(4500.00, IncomeTaxCalculator.calculateIncomeTax(20_000.00));
}

@Test
void calculateTaxExactlyAtBracket() {
    assertEquals(2000.00, IncomeTaxCalculator.calculateIncomeTax(10_000.00));
}
```

The report looks better now, but not perfect.

<table class="src">



<tbody><tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_1">
1
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_1"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">package test.pit;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_2">
2
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_2"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="uncovered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_3">
3
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_3"></a>
<span>
</span>
</span>
</td>
<td class="uncovered"><pre><span class="">class IncomeTaxCalculator {</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_4">
4
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_4"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_5">
5
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_5"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    private static final double INCOME_TAX_20 = 0.2;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_6">
6
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_6"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    private static final double INCOME_TAX_25 = 0.25;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_7">
7
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_7"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    private static final double INCOME_TAX_25_BRACKET = 10_000.00;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_8">
8
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_8"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_9">
9
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_9"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    public static double calculateIncomeTax(double income) {</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_10">
10
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_10"></a>
<span>
</span>
</span>
</td>
<td class="covered"><pre><span class="">        var incomeTax = 0.00;</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_11">
11
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_11"></a>
<span>
</span>
</span>
</td>
<td class="covered"><pre><span class="">        var remainingTaxableIncome = income;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_12">
12
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_12"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_13">
13
</a></td>
<td class="survived">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_13">2</a>
<span>
1. calculateIncomeTax : changed conditional boundary → SURVIVED<br>
2. calculateIncomeTax : negated conditional → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="survived">        if (income &gt; INCOME_TAX_25_BRACKET) {</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_14">
14
</a></td>
<td class="killed">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_14">1</a>
<span>
1. calculateIncomeTax : Replaced double subtraction with addition → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="killed">            var incomeOver25PercentBracket = income - INCOME_TAX_25_BRACKET;</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_15">
15
</a></td>
<td class="killed">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_15">2</a>
<span>
1. calculateIncomeTax : Replaced double multiplication with division → KILLED<br>
2. calculateIncomeTax : Replaced double addition with subtraction → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="killed">            incomeTax += incomeOver25PercentBracket * INCOME_TAX_25;</span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_16">
16
</a></td>
<td class="killed">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_16">1</a>
<span>
1. calculateIncomeTax : Replaced double subtraction with addition → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="killed">            remainingTaxableIncome -= incomeOver25PercentBracket;</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_17">
17
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_17"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">        }</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_18">
18
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_18"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class=""></span></pre></td></tr>


<tr>
<td class="covered">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_19">
19
</a></td>
<td class="killed">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_19">3</a>
<span>
1. calculateIncomeTax : replaced double return with 0.0d for test/pit/IncomeTaxCalculator::calculateIncomeTax → KILLED<br>
2. calculateIncomeTax : Replaced double addition with subtraction → KILLED<br>
3. calculateIncomeTax : Replaced double multiplication with division → KILLED<br>

</span>
</span>
</td>
<td class="covered"><pre><span class="killed">        return incomeTax + (remainingTaxableIncome * INCOME_TAX_20);</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_20">
20
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_20"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">    }</span></pre></td></tr>


<tr>
<td class="na">
<a name="org.pitest.mutationtest.report.html.SourceFile@2fac80a8_21">
21
</a></td>
<td class="">
<span class="pop">
<a href="#grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_21"></a>
<span>
</span>
</span>
</td>
<td class=""><pre><span class="">}</span></pre></td></tr>



<tr><td></td><td></td><td><h2>Mutations</h2></td></tr>

<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@2fac80a8_13">13</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_13"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> negated conditional → KILLED</p> <p class="SURVIVED"><span class="pop">2.<span><b>2</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>none</span></span> changed conditional boundary → SURVIVED</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@2fac80a8_14">14</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_14"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> Replaced double subtraction with addition → KILLED</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@2fac80a8_15">15</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_15"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> Replaced double multiplication with division → KILLED</p> <p class="KILLED"><span class="pop">2.<span><b>2</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> Replaced double addition with subtraction → KILLED</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@2fac80a8_16">16</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_16"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> Replaced double subtraction with addition → KILLED</p> 
</a></td>
</tr>
<tr>
<td><a href="#org.pitest.mutationtest.report.html.SourceFile@2fac80a8_19">19</a></td> 
<td></td>
<td>

<a name="grouporg.pitest.mutationtest.report.html.SourceFile@2fac80a8_19"> 

<p class="KILLED"><span class="pop">1.<span><b>1</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> replaced double return with 0.0d for test/pit/IncomeTaxCalculator::calculateIncomeTax → KILLED</p> <p class="KILLED"><span class="pop">2.<span><b>2</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> Replaced double addition with subtraction → KILLED</p> <p class="KILLED"><span class="pop">3.<span><b>3</b><br><b>Location : </b>calculateIncomeTax<br><b>Killed by : </b>test.pit.IncomeTaxCalculatorTest.[engine:junit-jupiter]/[class:test.pit.IncomeTaxCalculatorTest]/[method:calculateTaxBracket25()]</span></span> Replaced double multiplication with division → KILLED</p> 
</a></td>
</tr>

</tbody></table>

There's still a surviving *conditional boundary* mutant on line number 13.
In this case, the mutatation changes the condition to `income >= INCOME_TAX_25_BRACKET` but it has no effect on the outcome of the tests.
According to mutation testing rules, this is a potential gap in the tests but if we read the code, we can see that it cannot affect the end result because 25% tax is charged on everything that's above 10,000.
`income - INCOME_TAX_25_BRACKET` evaluates 0 when `income` is 10,000.

Sometimes, mutation testing can produce results where you, the author, have to evaluate whether it's significant.
It could be that the code under test is not needed or the code could be rewritten.
For example, in our case we could remove the if-sentence altogether and get a 100% green mutation test result with the following implementation.

```java
public static double calculateIncomeTax(double income) {
    var incomeTax = 0.00;
    var remainingTaxableIncome = income;

    var incomeOver25PercentBracket = Math.max(income - INCOME_TAX_25_BRACKET, 0);
    incomeTax += incomeOver25PercentBracket * INCOME_TAX_25;
    remainingTaxableIncome -= incomeOver25PercentBracket;

    return incomeTax + (remainingTaxableIncome * INCOME_TAX_20);
}
```

Use common sense.
Mutation testing cannot evaluate the readability or the performance of the code.
Arguably, having the if-sentence present makes it very clear that 25% tax is applied to everything above 10,000.
Additionally, the new implementation performs unnecessary work when `income <= 10_000`.

## Code Coverage Anybody?

When we try to understand the quality of our test suite, discussions often lead to measuring code coverage.
At first glance, there's a similarity between mutation testing and code coverage, especially when we look at the red and green lines of code in the mutation testing report.

Code coverage tells you which lines of code were executed during a test.
Technically, you can get 100% line coverage with a single test but that doesn't guarantee the test executed all possible code paths.
You need to specifically measure branch coverage if you're interested in all the possible decision points in the code.
What's more, you can get 100% line coverage even if you don't assert anything.
Therefore, measuring code coverage alone cannot protect you against poorly maintained tests.
But that doesn't mean code coverage is a completely useless metric though.
You need to understand what you're measuring.

## Is Mutation Testing the end-all be-all?

Mutation testing is an interesting way to discover gaps in our tests and a tool we can include into our toolbox.
However, like any tool, it comes with its own set of challenges.
Even on a relatively small codebase, the number of generated mutations can grow very high.
Now imagine a scenario where the entire test suite takes a minute to finish, and there are 1000 mutations.
Mutation testing would take approximataely 17 hours to finish.

Of course, there are optimisations that can be done:

* limiting the number of mutations generated for each run
* limiting the types of mutations included into each run
* excluding tests
* excluding code under test
* PIT has an [incremental analysis feature](https://pitest.org/quickstart/incremental_analysis/)

Due to the scalability issues, mutation testing is probably not something you want to run on your CI/CD pipeline on every commit.
Instead, consider running it on a schedule automatically or you could run it locally on a smaller scale, only on the source files you're working with.

## Summary

Mutation testing is a method for testing the quality of the tests.
It works by making small changes, or *mutations*, in your code to see if your tests can find the errors.
At first glance, it's similar to measuring code coverage, which tells you how much of your code is tested, but mutation testing also checks how well your tests work.
These two methods are different and complement each other.
Remember, the aim isn’t to kill all mutants but to make your tests better.