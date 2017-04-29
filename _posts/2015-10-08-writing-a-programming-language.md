---
title: Writing a (garbage) programming language
layout: post
permalink: writing-a-programming-language
published: true
---
Let me introduce you to my friend *FOG*. *FOG* stands **F**ormatting **O**riented **G**arbage, but *FOG* likes its short name. *FOG* is an esoteric programming language that uses formatting to denote control structures and intentions. More specifically it can interpret a file using a highly opinionated subset of HTML4 as a computer program. I have been working on *FOG* on and off for over a year now and would like to share some information about our process and the language so far.

## Why?
*FOG* started out as my final project for *AP Computer Science A*. The purpose of *FOG* for me was to learn about managing standards and versioning and how to write a compiler. During the design phase we had three students working on documents and specifications. It was a great experience. Beyond that, *FOG* is cool, it makes code easy as pie to render and demonstrate, and you can be sure that all that is being rendered is your ideas, not distracting syntax. In FOG, you can type comments anywhere and use any characters (even HTML tags if you use `<pre>`), it is perfect for showing off workflows. 

## Documenting first; coding second
When I decided to start building *FOG*, I started by making a solid documentation framework. It was mostly blank, but it made it clear to contributors and the public what I wanted the language to be and where it was so far. Despite my careful planning, this failed me. The initial idea was to use the *RTF* format, but as we talked, *HTML* sounded better:

* HTML allows you to use a GUI or just type markup.
* HTML has a system for executing scripts
* HTML has a styling system which developers could leverage

But *HTML* wasn't as structured or opinionated as *RTF* and that created a hurdle for us to jump over before we started documenting constructs.

## Removing "fog"
We now had to deal with the complexity of HTML and it can be really complicated In HTML I can create an underline in a virtually "infinite" amount of ways

```html 
<u>underline</u>
```

```html 
<ins>underline</ins>
```

```html 
<span style="text-decoration:underline">underline</span>
```

```html 
<style>
.line-under{
  text-decoration: underline;
}
</style>
<blink class="line-under">underline</blink>
```

> How could we parse all of this?

The answer was simple: **we couldn't**. Not without writing a bunch of useless code and throwing performance to the dogs.

To solve this issue, we created a compile step called dfog. We explain dfog to people as a sort of "linker", that doesn't quite do it justice. dfog will take any HTML code and turn it into a strict subset of HTMl for a FOG compiler to use. It does this by spinning up a WebKit instance and letting all the JavaScript and CSS be interpreted there. It then extracts the code in a format that the FOG compiler can use. dfog has the pleasant side-effect of allowing developers to modify their FOG code using JavaScripts and CSS files. 

## Now to create the specification



### Structure of FOG
![Fog structure](http://i.imgur.com/oMs9cUf.png)

* Red represents a specification or a document used in creating a specification.
* Blue represents code independent of the *FOG* source. 
* Green represents programs in the *FOG* ecosystem. 
* White shows what code could look in certain steps of the compile process.

### Garbage allocation
We realized quickly that HTML had a limited amount of tags and those tags had limited support. Making sure features didn't clash was a challenge. To help with this we created a repository for allocating HTML "tags to specific language functions.

<script src="https://gist.github.com/Falkirks/3598fdae4ded3c85bffd54f69a3cee15.js"></script>

From this repository we hoped to produce our specifications for language versions.

### Simple incremental versions 
The FOG language specification would be implemented in simple incremental versions. Starting at `0` with the least features. We created a repository for managing these versions and providing examples of how to use their features. As of yet, we are still on version `0` :P

You can look at that [on GitHub](https://github.com/foglang/language-versioning/tree/master/0).

### Trying to write a compiler...
The initial idea was to compile into C++ with some standard libraries and then run through the LLVM and get optimizations. One of the members of the team wrote a syntax tree generator in C for FOG (https://github.com/foglang/cfog/blob/master/main.c).

We then looked at compiling in Python into Python. We got a bit further with that (https://github.com/foglang/pyfog/blob/master/main.py).

I decided a while after that it would be nice to just have something anything to worked to play around with. So I wrote a barebones compiler for FOG `0` in JavaScript (https://github.com/foglang/wisp) . It doesn't do anything, but print stuff.

So there isn't really a compiler, you could write one :P

## What I got out of this?
Through this project I gained valuable experience managing a set of standards and developing a language. 

If you are interested in learning more about FOG. You can check out the documentation at http://fog.falkirks.com. Or look at our code at https://github.com/foglang.

