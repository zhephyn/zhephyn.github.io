---
layout: post
title:  "How to prevent name collisions when naming your Ruby classes"
date:   2025-07-10 15:49:47 +0300
categories: ruby
---

## Problem: 
For whatever reason, you want to create a class named ```Integer``` and instantiate objects with that class. 

## Approach:
You create the class like this:
```ruby
3.2.6 :001 > class Integer
3.2.6 :002 > end
 => nil 
```
That works like a charm. Now you proceed to instantiate objects using the newly created ```Integer``` class.
Boom, you run into the error below:
```ruby
3.2.6 :003 > Integer.new
(irb):3:in `<main>': undefined method `new' for Integer:Class (NoMethodError)

Integer.new
       ^^^^
	from /usr/share/rvm/gems/ruby-3.2.6/gems/irb-1.15.1/exe/irb:9:in `<top (required)>'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/irb:25:in `load'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/irb:25:in `<main>'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/ruby_executable_hooks:22:in `eval'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/ruby_executable_hooks:22:in `<main>'
3.2.6 :004 > 
```
## Explanation of why we run into the error:
While this is valid ruby syntax for instantiating objects from a class, you can see we ran into a specific kind of error named a "```NoMethodError```".

Ruby ships with an inbuilt class called ```Integer``` and this class doesn't have a ```new``` method defined for it. When you try to create new objects by calling ```Integer.new ```, instead of Ruby calling the ```new``` method on your newly created ```Integer ``` class, the ```new``` method is called on ruby's inbuilt ```Integer``` class. And because this class doesn't have a method named ```new``` defined for it, we run into the "no method error". 

```ruby
<main>': undefined method `new' for Integer:Class (NoMethodError)
```

In simple terms, the error loosely translates to: "**Hey, you called a method named ```new``` on my ```Integer``` class. But when i go through my list of methods defined for the ```Integer``` class, I can't find this method named ```new```.**"

A simple irb check lets you know that indeed this default ```Integer``` class has no method named ```new``` defined for it.
```ruby
3.2.6 :007 > Integer.methods.include?(:new)
 => false 
3.2.6 :008 > 
```
## Potential solution(s):
One solution is to consider a different name for your new class. Ideally one that doesn't conflict with the names of inbuilt ruby classes. That could work, but what if you don't wanna consider a different name?

In that case, you can solve the above error by wrapping your newly created class in a module. You can name that module what ever you like and use it to reference the custom ```Integer``` class when creating new objects like below:

```ruby
3.2.6 :008 > module Namespace
3.2.6 :009 >   class Integer
3.2.6 :010 >   end
3.2.6 :011 > end
 => nil 
3.2.6 :012 > Namespace::Integer.new
 => #<Namespace::Integer:0x000075c9a235a098> 
3.2.6 :013 > 
```
As you can see, with the module approach, we don't run into any problems instantiating objects with the custom ```Integer``` class.

## Bonus:
The simple irb check below confirms that the ```new``` method is available for your newly created ```Namespace::Integer``` class. 

```ruby
3.2.6 :013 > Namespace::Integer.methods.include?(:new)
 => true 
3.2.6 :014 > 
```
In case you hadn't noticed I named my module "Namespace". This is because the act of wrapping a class in a module to prevent name collisions with another class, for example the inbuilt ```Integer``` and custom ```Integer``` classes, is referred to as **Namespacing**. Therefore, my module name was a subtle hint to the term used to refer to the technique I was applying to solve the problem. 