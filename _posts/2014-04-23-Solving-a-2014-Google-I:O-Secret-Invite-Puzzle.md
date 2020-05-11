---
layout: post
post_number: 16
title: "Solving a 2014 Google I/O Secret Invite Puzzle"
excerpt:
  I didn't win the Google I/O ticket lottery this year, but thought I'd try to find and solve one of the secret puzzles Google was hiding, to get a chance to buy a reserved ticket.
---

![Android logo](/assets/posts/16/Android.png){: .image-left }
I didn't win the Google I/O ticket lottery this year, but thought I'd try to find and solve one of the 
secret puzzles Google was hiding, to get a chance to buy a reserved ticket.

I noticed that the Google Developers page header contained the text "I/O", with different colors 
for each of the letters. Refreshing the page changed their colors, and occasionally removed them
from the page. Looking at the source code, I found this:

{% highlight html %}
<span style="color: #386834">I</span>/<span style="color: #317368">O</span>
<!-- Still need a hint? http://goo.gl/Eypmp -->
</li><!-- 110101 110010 110001 110011 110100 110000  -->
{% endhighlight %}

You might as well check out the hint [url](http://goo.gl/Eypmp) before reading on. Yep, it got me too. 
But... it turns out it *is* a hint.

The blocks of numbers in the comments on the last line were clearly binary, so I converted them to 
integers: 5,2,1,3,4,0. I refreshed the page several times, printing out lots of values for the hex color 
of the "I" and "O", and with the resulting binary->integer conversions. I didn't see any real pattern in 
the hex color values, but did notice that each of the binary->integer conversions always had exactly one 
of each 0,1,2,3,4,5 - this looks like a series of indexes.

Going back to our hint, I realized that the goal is to come up with the 6-character code to use as a 
Google shortened URL. Rather than 6 characters, it appears that we have 12 and a series of 6 indexes, 
but really, those color codes each have 3 hex components:

{% highlight text %}
0x38, 0x68, 0x34, 0x31, 0x73, 0x68
{% endhighlight %}

[Convert these to decimal](https://www.mathsisfun.com/binary-decimal-hexadecimal-converter.html):

{% highlight text %}
56, 104, 52, 49, 115, 104
{% endhighlight %}

And now, convert them from decimal to ASCII characters:

{% highlight text %}
8, h, 4, 1, s, h
{% endhighlight %}

Let's apply the indexes (5,2,1,3,4,0) by rearranging these letters so that the 5th 0-based index character is 
first, then the 2nd, 1st, 3rd, 4th, and 0th:

{% highlight text %}
h4h1s8
{% endhighlight %}

Prefix that code with the Google URL-Shortening domain, and we have: 

{% highlight text %}
http://goo.gl/h4h1s8
{% endhighlight %}

Success! Well, sort of. By the time I solved this, all of the codes seemed to be taken. Of hundreds of codes 
generated, only a few worked. I'm not sure if some random ones were thrown in, or if I'm missing another piece 
to the puzzle. In any case, the shortened link redirects you to a fun console-based game, reminiscent of the 
old terminal-based Zork and Hitchhiker's Guide to the Galaxy games, but with color and simple animations. 

At the time of this writing, this game link still works, so have a go at it.  Take a look at my Python code 
that generates URLs by refreshing the Google Developers page and calculating the secret shortened URL.

TLDR: [Game Link](http://goo.gl/h4h1s8), [Python Code](https://gist.github.com/wblakecaldwell/11203637)

Here's a screenshot with all of the different options selected.

![Google I/O Game](/assets/posts/16/Google_IO_Puzzle_Screenshots.png)
