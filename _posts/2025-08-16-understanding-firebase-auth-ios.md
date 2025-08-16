---
layout: post
title: Understanding Auth.auth() in Firebase
subtitle: Why Do We Have to Repeat the Words Twice
tags: [ios]
---

In getting started with Firebase Authentication in iOS, the way you can handle most of the authentication processes uses `Auth.auth()` as indicated in the snapshot below:

<div align="center">
  <img src="/assets/images/from_posts/img-1-2025-08-03-understanding-firebase-auth-ios.png" alt="">
</div>

It seems redundant (at least initially). If you read it, it seems like it is calling Auth twice to do its job, but I think most people would understand it once they searched for the answers, which I did and will share in this writing.

## What the Documentation Has to Say

Well, if you look at the documentation, it has no resources that explain why `Auth.auth()` is the convention. So I suspect that this convention is "*internally understood*," but I don't understand it yet, which is why I want to learn more about it.

The first step is understanding what does `Auth` mean.

<div align="center">
  <img src="/assets/images/from_posts/img-2-2025-08-03-understanding-firebase-auth-ios.png" alt="">
</div>

It says here that `Auth` is a class and `auth()` is a method of that class that gets the auth object, as shown in the definition. 

And that's actually it; seems easy enough!

## What Xcode Has to Say

Let's evaluate this working code by trying to remove the `Auth` class

<div align="center">
  <img src="/assets/images/from_posts/img-3-2025-08-03-understanding-firebase-auth-ios.png" alt="">
</div>

And that's the key! 

<mark><b>By using auth() only, Xcode has no idea where to find it.</b> Because remember that in most libraries that you would use in Swift language, <b>every method lives inside a construct, either in protocols, structs, or classes.</b></mark>

What I suspect in this one is that it tries to search globally by default if the `auth()` function exists. It seems that by importing `FirebaseAuth`, it does not register its methods in the global function scope to be used implicitly.

The use of the `Auth` class prefix makes it so that when you write `Auth.auth()`, Swift can recognize the `Auth` class from the  imported `FirebaseAuth` library, and execute the `auth()` method that resides within that class.

<div class="conclusion-divider">
    <hr>
</div>

There you have it! It's very easy to understand once you think of it that way, but it will not always be obvious if you are just starting out.