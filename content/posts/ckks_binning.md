---
title: "Binning in FHE with CKKS encoding"
date: 1970-01-01
draft: false
ShowToc: true
author: Herocharge
---

### Introduction

Fully homomorphic encryption (FHE) let's you perform computation on encrypted data. There are many different FHE schemes. Along with the encryption schemes, there is usually an encoding scheme which can take numbers in one space and turn them into numbers which can be encrypted using the encryption scheme. One such encoding scheme which can encode complex numbers is CKKS.  

I will not get into the details of how FHE internals work. For the purposes of this blog, I will be using a nice abstraction provided by the Microsoft SEAL library. In fact, I will be using a python wrapper of the library called tenseal.

### Premise

In the FHE space, we can perform two operations on encrypted numbers - addition and multiplication. We can add and multiply constants as well. 

```
If:
a, b -> encrypted real numbers
x -> regular real numbers

Allowed operations:
- a + b
- a * b
- a * x
- b + y

```

One should also keep in mind that we are constrained on the depth of operations that we can perform. Every operation adds a bit of noise to the result, making it less precise. If we have too much computational depth, the result becomes too erroneous and decrypting it will result in garbage value.

```
For example,
a, b, c -> encrypted values

z = a + b # Computation depth = 1
y = z * c # Computation depth = 2
x = y + a # Computation depth = 3

```

Multiplication adds significantly more noise than addition. So I would mostly worry about the multiplication depth of the algorithm that I will be implementing.


### Problem statement

Given the above constraints, I want to come up with an algorithm which takes an encrypted value (say `X`), an unencrypted range `[a, b]` and the number of same-sized bins the unencrypted range is divided into `n`. And gives as output which bin `X` lies in.

To rephrase: Given consecutive bins `[a0, a1]`, `[a1, a2]`, ... ,`[a(n-1), an]`. Produce a one-hot vector of length `n`, where the element corresponding to the bin containing `X` is `1` and the remaining elements are `0`.

```
Example 1:
Input:
 X = 15 
 bins = [[1, 10], [10, 20], [20, 30]]
Output:
 [0, 1, 0]

Example 2:
Input:
 X = 35 
 bins = [[1, 10], [10, 20], [20, 30]]
Output:
 [0, 0, 0]
```

### Idea 1
The most straightforward way to solve this problem is to loop over the bins and checking if there the input number lies in that bin.

```py
def idea_1(X, bins):
    one_hot = [0 for _ in bins]
    for i, bin in enumerate(bins):
        if bin.lower < X and X < bin.higher:
            one_hot[i] = 1
```

There are several challenges with implementing this algorithm in the FHE setting.

Firstly, we have a conditional statement (`if`) which uses the encrypted input `X` as part of calculating it's boolean value. This causes problems because since X is encrypted, the result of the operations ` < X` and ` > X` will also result in encrypted boolean values. If the resulting boolean value was not encrypted, then that would mean information about the encrypted value is leaked, but we cannot have that, so we define `<` and `>` operations in a way that it returns encrypted booleans when using encrypted input.  

This constraint prevents us from using branching totally. So regardless of the boolean value the code inside the if-block needs to execute the code inside it. However we can still preserve the correctness of the program by multiplying the encrypted boolean value with the changes made inside the if-block to variables.

```py
def idea_1_no_if(X, bins):
    one_hot = [0 for _ in bins]
    for i, bin in enumerate(bins):
        update_value = 1
        boolean_value = bin.lower < X and X < bin.higher
        one_hot[i] = boolean_value * update_value
```

Now we don't have to worry about branches!

Another challenge with this is the actual comparison part. We need to implement `<` and `>` functions, but if you recall we can only perform addition and multiplication, so we need a way to approximate these comparisons.

Comparison in regular code is quite inexpensive, but in the world of FHE, it gets really expensive. The algorithms which implement comparison often resort to taylor series expansion or some other polynomials. Unfortunately, if we want precise comparison we need to use higher degree polynomials which would increase our multiplication depth, thereby adding noise.

```
TODO:
- Add links to papers and blogs
- Add the algorithm that uses gaussian
```