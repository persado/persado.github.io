---
layout: post
title: NaN and Infinity in Ruby
excerpt: As most well-behaved programming languages, Ruby handles a zero division (1 / 0) by raising an error and, in doing so, alerts you of your grave mistake with a bold, in-your-face, ZeroDivisionError. However, when either dividing a float value by zero, an integer by float zero (0.0), or dividing a float value by float zero, there is no error...
author: tim_fraczak
---

As most well-behaved programming languages, Ruby handles a zero division (1 / 0) by raising an error and, in doing so, alerts you of your grave mistake with a bold, in-your-face, ZeroDivisionError. However, when either dividing a float value by zero, an integer by float zero (0.0), or dividing a float value by float zero, there is no error. When doing these division operations, the resulting values are:

```
( 1 / 0.0 || 1.0 / 0 || 1.0 / 0.0 )   # => Infinity
( -1 / 0.0 || -1.0 / 0 || -1.0 / 0.0 )  # => -Infinity
0 / 0.0 || 0.0 / 0 || 0.0 / 0.0   # => NaN
```

I found this interesting because I didn’t realize Infinity and NaN were even a thing in Ruby. Furthermore, they aren’t standalone classes like nil, but constants on the Float class (Float::INFINITY and Float::NAN). Also, they’re both truthy! Unlike in JavaScript where NaN is falsey.

So, why does this matter? Well, we can do further operations on these values without error. And resulting comparison values don’t always make the most sense intuitively. 
Here’s what makes sense to me:

```
Infinity > -Infinity || num < Infinity || num > -Infinity  # => true
```

Aaaannnd here’s what doesn’t make sense (at least to me it doesn’t):

```
Infinity > NaN || NaN == NaN   # => false
( 2.0 / 0 ) > ( 1.0 / 0 )   # => false
( 87634.0 / 0 ) == ( 0.0000001 / 0)   # => true
```

...wut?! One Infinity can’t be greater than another, even though one increases faster than another? That’s basic mathematics! Also... NAN IS NOT EQUIVALENT TO ITSELF?? Ruby, you must be a distant cousin of our dear, beloved JavaScript...
Anyway, why this matters: this means that we get actual return values which can be further evaluated, and can give especially confusing results for comparisons. So, that could potentially be a problem in writing specs for our codebase, or worse, edge cases that weren’t tested before pushing code to production!

**Example:** in one of our specs, we test a service which calculated percentages which were most certainly represented by float values, and we calculated the denominator using `model.other_models.count`, however, all of these models were build_stubbed which means that none of these things exist in the test DB. 

**Problem:** `#count` does a SQL query to fetch what it needs to count, but since none of this stuff exists in the DB, the return value for the denominator is a big ol’ ZERO. Now, we are doing calculations like ( 0.5 / 0 ), and let’s say our test is something like:

```
expect(resulting_value <= 0.5).to be(true)
```

What we think is happening is something like:

```
( 0.5 / 2 ) <= 0.5
```

which should be true, but what we’re actually doing:

```
( 0.5 / 0 ) <= 0.5
```

and this returns false, because it evaluates to Infinity <= 0.5. We don’t see any errors, we don’t see what exactly is going wrong, and worse yet, say we’re not directly testing the comparison itself but it resides somewhere in the chain of code that is executed to get us the result we’re testing, and it’s returning the wrong value. This is incredibly frustrating because there’s no back trace, there’s no error thrown, nothing to indicate where our code is going wrong.

### Conclusion

I believe Ruby is a distant cousin to JavaScript, in that Ruby can get a little whacky when trying to evaluate certain operations with float values. All jokes aside, I learned NaN and Infinity are actually required values for floats in most programming languages specified by the [IEEE 754-1985 standard](https://en.wikipedia.org/wiki/IEEE_754-1985#Representation_of_non-numbers) to handle invalid results of a computation. Moreover, if there’s anything I think you should pull away from this is that if you have float computations/comparisons in your code, your specs are failing, and there seems to be no other explanation as to why a comparison is returning false when you know deep within your own heart that it’s supposed to be true, you might have zero division with floats.
