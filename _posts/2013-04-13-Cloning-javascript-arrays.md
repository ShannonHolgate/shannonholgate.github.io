---
layout: post
title: "Cloning Javascript arrays"
description: "A short explanation on how to clone Javascript arrays"
category: articles
tags: [Javascript]
comments: true
---

<figure>
	<img src="/images/star-wars-clones-large.jpg">
	<figcaption>Clones, Clones everywhere.</figcaption>
</figure>

I found myself researching this quite recently when faced with a bit of a random problem in my current project.
My magical problem first appeared when I changed an element in one array which changed the other array which I had "cloned" using the *newArray = oldArray* method below.

### What I thought

<pre><code class="language-javascript">/**
* How NOT to do it
**/
var carArray = ["Fiat","Vauxhaull","Toyota"];
var newCarArray = carArray;</code></pre>

It turns out that this only copies the pointer to the old array, which makes sense now that I've actually found out what was wrong (hindsight is a wonderful thing).

### The truth

To get things right we should explicitly clone the array. There are a few different approaches that many web developers take to this but the simplest uses the *Array.splice()* function. *Array.splice()* is used to delete elements at certain indices in an array, we can leverage this to return an entire array by simply passing 0 as the first argument. 0 works because there is nothing at index 0, so the full array is returned.

<pre><code class="language-javascript">/**
* How to do it quickly and correct
**/
var carArray = ["Fiat","Vauxhaull","Toyota"];
var newCarArray = carArray.splice(0);</code></pre>

Keep in mind, if your array has objects in it, the objects are not copied only the references are copied.
That's all there is to it! No need to mess about with iterating through your arrays, just one simple line.

Give this article a share if it helped you out and feel free to add me to your professional network!