---
layout: post
title: How forms should be done in ZF2/3
---

Zend Forms are great and flexible, however, even the official documentation does not show how to make them properly. Too many times had I seen people having trouble creating simple forms or form elements.
As the documentation is lacking and the number of real world examples on the Internet is scarce, I have released a Zend Framework 3 playground app.

Currently it contains a form, which contains other elements like:
* simple e-mail input
* csrf hidden input
* a fieldset
* a dropdown with dynamically populated values
* a dedicated Select2 dropdown

There are a number of comments in the code where I explain what's going on under the hood.

The app is Docker based and easy to run with a couple of simple commands. I published it at Github recently. Just head over to [https://github.com/mikemix/zf3-form-example](https://github.com/mikemix/zf3-form-example) and have fun. I plan on extending the form with more elements so stay tuned.
