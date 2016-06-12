---
layout:     post
title:      "C++: Yet Another Kind of error handling"
subtitle:   "Self document your code and let users decide how to handle error"
date:       2016-06-11 10:12:34
author:     "Nicolas DI PRIMA"
tags:       [ Haskell, Rust, Error handling ]
comments:   true
header-img: "img/post-bg-01.jpg"
---

I have been a C++ developer for a little while, I have seen different methods
to provide meaningful API:

* the **C-like**: using error code and returning output in the parameter;
* the **exception-like**: using exceptions.

The difficulties are to:

* guarantee that the error handling is documented;
* ensure it does not change (requires a lot of regression tests);
* keep undefined behaviours as "exceptional".

That's it: exceptions are for exceptional situations. And again, it is not
easy to provide up to date documentation of the kind of exceptions
your function may throw.

So, let's try something else. Let's see what other languages have done to
try to solve the issue of non self documented APIs.

# Haskell

Certainly my favourite language. In Haskell there are different methods
to return error information to the user. The simplest
is **Either** or **Maybe**.

{% highlight Haskell %}
-- define an error type
data Error = IOError
           | UTF8Error

-- just provide the Function signature to show you how
-- self documented it is
readFile :: FilePath -> Either Error Text
{% endhighlight %}

So the function is named: **readFile**, it takes a **FilePath** as parameter
and it returns either an **Error** or a **Text**.

This design provides you with some information about the kind of error handled
by this function. I don't say this function won't throw an exception: but the
signature of the function tells you what you can expect.

Now, you know how you can use it:

**If you want to interrupt your program now:**

{% highlight Haskell %}
main = do
   case readFile "document.txt" of
      Left err -> error $ "error in readFile: " ++ show err
      Right txt -> printLn txt
{% endhighlight %}

**If you want to fail quietly**

{% highlight Haskell %}
main = do
   either
      (return ())
      print
      (readFile "document.txt")
{% endhighlight %}

# Rust

The latest language I have been looking into. Full of
[promise](http://groveronline.com/2016/06/why-rust-for-low-level-linux-programming/)
for the System Developers.

The error management is also very simple and self documented:
[Result](https://doc.rust-lang.org/std/result/enum.Result.html).

{% highlight rust %}
enum Error {
  IOError,
  UTF8Error
}

fn readfile(f: Path) -> Result<Error, String>;
{% endhighlight %}

And now you can, the same way you would do in Haskell:

**throw the error**

{% highlight rust %}
// unwrap() will throw an exception (panic!)
let txt = readFile("document.txt").unwrap();
println!("{}", txt);
{% endhighlight %}

**or fail quietly**

{% highlight rust %}
// only execute the lamba in the function succeed
readFile("document.txt")
  .and_then(|txt| println!("{}", txt));
{% endhighlight %}
