---
layout: post
title: How to enable Twig template autocompletion with Symfony integration plugin in phpStorm
---

The plugin currently is lacking support for the new Symfony's directory structure 
since we keep Twig files in the `templates/` directory.

The workaround is very simple, one just has to add the missing directory to the list of known Twig paths:

![Adding Twig path](/images/2018-11-01-phpstorm-symfony-plugin-twig-template-autocompletion.jpg)

That's all, enjoy:

![Adding Twig path](/images/posts/2018-11-01-phpstorm-symfony-plugin-twig-template-autocompletion-2.jpg)
