---
layout: post
post_number: 17
title: "Just for Fun: Hash Table Implementation in C"
categories: [C, Algorithms]
---

[Hash tables](http://en.wikipedia.org/wiki/Hash_table) are cool. Like many developers, I used them for years 
without giving much thought to how they actually work. Just for fun, I decided to look into it - and while 
I was at it, brush up on [C](http://en.wikipedia.org/wiki/C_(programming_language)), which I hadn't spent 
much time with since my first internship years ago.

[TLDR: Take a look at the code on GitHub](https://github.com/wblakecaldwell/fun-with-c/tree/master/hash-table)

## Example: Hash Tables in Go

Hash tables (also known as "hash maps", or "maps") are so useful that they're a 
[built-in type](https://blog.golang.org/go-maps-in-action) in the 
[arguably](http://golang.org/doc/faq#generics) minimalist 
[Go programming language](https://golang.org). They allow you to create a simple lookup with a key/value pairing.

For example, if you forget which Wiggle wears which color, you could create a map of color (key) to Wiggle name 
(value), then refer to them by color through the map ([try it in the Go Playground](http://play.golang.org/p/BjXx4WDcT0):

{% highlight golang %}
// map of color -> wiggle (don't judge.)
wiggle := make(map[string]string)
wiggle["purple"] = "Jeff"
wiggle["red"] = "Murray"
wiggle["blue"] = "Anthony"
wiggle["yellow"] = "Greg"

fmt.Printf("Purple: %s\n", wiggle["purple"])
fmt.Printf("Red: %s\n", wiggle["red"])
fmt.Printf("Blue: %s\n", wiggle["blue"])
fmt.Printf("Yellow: %s\n", wiggle["yellow"])
{% endhighlight %}

## What are Hash Tables Doing?

Conceptually, hash tables are pretty simple. Under the hood, a hash table has a bunch of buckets for its 
values. The hash table knows in which bucket to find a value by its key, so it can quickly find anything it's 
looking for. It's much like how you'd keep track of your old baseball cards in boxes. You have a system - in 
this case, a box per year. You have 50 boxes, but when you're looking for a specific card, you jump right 
to the correct box. Once you open that box, you rummage through it until you find the right card. As the 
number of buckets increases, hash table lookups approach 
[constant time, O(1)](http://en.wikipedia.org/wiki/Time_complexity#Constant_time).

## But... How Do They Work?

A hash table references a fixed number of buckets which can be represented as an array of simple 
[linked lists](http://en.wikipedia.org/wiki/Linked_list). As we give the hash table a value to store, the key 
is fed through a [a hashing algorithm](http://en.wikipedia.org/wiki/Hash_function) which outputs a long 
integer called that key's "hash code". We [mod](http://en.wikipedia.org/wiki/Modulo_operation) that hash code by 
the number of buckets to figure out which bucket to use, then add the key and value to the end of the linked list 
in that bucket, creating the list if it doesn't yet exist.

For example... Say you have 10 buckets with indexes 0 through 9, and a key with hash code equal to 223. 
[223 mod 10](http://www.wolframalpha.com/input/?i=223+mod+10)"> is 3, so we'll append the key/value to the 
linked list in bucket with index 3. If you had 1,000,000 buckets, then you'd append the key/value to the linked 
list in bucket with index 223, since [223 mod 1000000](http://www.wolframalpha.com/input/?i=223+mod+1000000)
is 223.

When you search for a value in a hash table by key, the same hashing algorithm is used on the key to figure out 
which bucket to look in. That bucket's linked list is traversed until the key is found, using an equality 
algorithm.

## Java: hashCode() and equals()

If you've worked with Java, you've probably implemented 
[hashCode()](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode()) a bunch of times, 
or more likely, copied and pasted from somewhere. You've no doubt heard that it's 
[important to implement it careflly](http://eclipsesource.com/blogs/2012/09/04/the-3-things-you-should-know-about-hashcode/),
but your program worked just fine with whatever you implemented.

How are these methods used in hash tables, and why are they important? Great questions! If you're using one of 
your custom types for a hash table's key, then your `hashCode()` method is used to determine which bucket to 
look in. Once the bucket is found, the match is determined with your `equals()` method, given the input key 
and keys in the bucket's list.

A poor implementation of `hashCode()` returns the same value for every input, reducing a hash table to a 
single bucket: a linked list. We'd lose all the performance benefits we were looking for with this data 
structure. Lookups go from 
[constant time (O(1))](http://en.wikipedia.org/wiki/Time_complexity#Constant_time) to 
[linear time (O(n))](http://en.wikipedia.org/wiki/Time_complexity#Linear_time). Implementing either improperly 
returns invalid results. You could store something in a hash table with a key, then get the wrong value, or 
no value at all when you try to retrieve it with the same key.

## Hash Table Implementation in C

I suppose it's a little late in the post to finally get to my implementation, but without an understanding of 
how a hash table works, the code is just a bunch of marks on your screen. In my implementation, both the keys 
and values are strings, but it wouldn't be too much work to make it more flexible.

My two custom types are the [hashTable](https://github.com/wblakecaldwell/c-hashtable/blob/master/hashtable.h#L31-L35) and the [linked list](https://github.com/wblakecaldwell/c-hashtable/blob/master/hashtable.h#L22-L28)
that represents each bucket:

{% highlight c %}
// linked list
struct hLinkedList
{
    char *key;
    char *value;
    struct hLinkedList *next;
};

// hashtable
struct hashTable
{
    unsigned int size;
    struct hLinkedList **lists;
};
{% endhighlight %}

Don't let `struct hLinkedList **lists;` scare ya - it's just an array that's sized and allocated later, when 
we determine how many buckets we need.

Here's my hashing function:

{% highlight c %}
int hashCode(char *str)
{
    int hc;

    hc = 0;
    while('\0' != *str)
    {
        // for each character, multiply the current hashcode by 31 and add the character's ascii value
        // multiply by 31 is the same as left shift by 5 and subtract value
        hc = hc << 5;
        hc = hc - hc + *str;

        str++;
    }
    return(hc);
}
{% endhighlight %}

This uses the traditional multiplication by 31 scheme, which, by being prime, 
[helps protect against losing information if the multiplication overflows](http://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier).

Since we're dealing with string functions, we use 
[strcmp](http://pubs.opengroup.org/onlinepubs/009695399/functions/strcmp.html) for the equality algorithm.

With my comments, and an understanding of C, the rest of the code should be fairly straightforward.

## Demo

As described in [README.md](https://github.com/wblakecaldwell/c-hashtable/blob/master/README.md):

Compile and execute the binary to run the demo use case in main.c:

{% highlight text %}
gcc *.c
./a.out
{% endhighlight %}

You should see:

{% highlight text %}
Adding 'hello'=>'world', 'color'=>'blue' and printing:
-----
hello -> world
color -> blue


Changing 'hello' value to 'goodbye', then printing:
-----
hello -> goodbye
color -> blue


Removing 'hello' and printing:
-----
color -> blue


Removing 'color' and printing:
-----
{% endhighlight %}

## Read My Codez!

Take a look at the [source code on GitHub](https://github.com/wblakecaldwell/fun-with-c/tree/master/hash-table)! 
Try modifying it to use a different type for either the keys or values.

Have your own fun!
