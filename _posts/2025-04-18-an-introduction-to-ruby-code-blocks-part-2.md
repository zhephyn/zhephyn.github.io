---
layout: post
title:  "An Introduction To Ruby Code blocks Part 2"
date:   2025-04-18 01:18:00 +0300
categories: ruby
---

## Shortcomings of Implicit Code Blocks
In Part 1 of this series, examination of the shortcomings of  Implicit code blocks in Ruby highlighted the fact that, they can't be stored and therefore manipulated or passed around to be run later. They have to be executed instantly and inside the method to which they are passed. 

Consider these 2 methods below both of which are meant to solve the same problem, printing someone's full name:
```ruby
def full_name
  first = "John"
  last = "Doe"
  yield(first, last) if block_given?
end

full_name do |first_name, last_name|
  "#{first_name} #{last_name}"
end

#OUTPUT
"John Doe"
```

```ruby
def full_name(&first_name)
  firstname = first_name.call
  fullname = firstname + " " + "Doe"
end

full_name do 
  "John"
end

#OUTPUT
"John Doe"
```

The first solution(method) employs yield and specifically the concept of Implicit code blocks in Ruby to solve the problem while the second utilizes Explicit code blocks to solve the same problem. For this exact problem, both these solutions are okay. However for use cases when we need to "*capture*" and "*save the code block for later execution*", the limitations of solution 1 which utilizes Implicit code blocks become evident very quickly and we'll see that the second solution i.e. the one employing explicit code blocks is better for the reasons highlighted below.
## Comparison between Explicit and Implicit Code Blocks in Ruby

| Aspect                                    | Implicit Code blocks(yield version)                     | Explicit code blocks(Proc version)                                                                                |
| ----------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Can the block be stored?                  | No. Can only be used inside the method                  | Yes. Can be stored as a Proc object.                                                                              |
| Can the block re-used outside the method? | No. Only called within the method                       | Yes. Since explicit code blocks can be stored as Proc objects, these Proc objects can be used outside the method. |
| Can the block be passed to other method?  | No. Only associated with the method to which its passed | Yes. Explicit code blocks captured as Proc objects can be passed to other methods.                                |

## Behind the scenes of how Ruby handles Explicit code blocks

Lets modify the fullname method a bit:
```ruby
def full_name(&first_name)
  puts first_name.class
  firstname = first_name.call
  fullname = firstname + " " + "Doe"
end
full_name do 
  "John"
end

#OUTPUT
Proc
"John Doe" 
```

When we pass a block to the above method, we can see that on checking for the class of the "first_name" object, the output is  "Proc" signifying that this is a Proc object. Here's a step by step flow of how that happens:

- Explicit code block passed to the full_name method is converted into a Proc object.
- Proc object is captured and assigned to and stored in the first_name parameter
- Inside the method, we call the Proc using:
```ruby
first_name.call
```
The above line executes the code block and returns the result of executing the explicit code block we passed to the method initially.

> Call is a ruby method defined for Proc objects and since first_name is a Proc object, we can therefore call the "call" method on it. 

- We assign the result of executing the explicit code block to a variable named "firstname"
- Using string concatenation, we add a string named "Doe" i.e the second name, to finally get the full name which is returned to us in the terminal. 
The conversion of an explicit code block into a Proc object is the reason for the flexibility associated with Explicit code blocks in ruby. Since we end up with a Proc object, we can use this object for all sorts of things from passing it to other methods as well as executing it at a later point in time. 

## The Importance of the Ampersand(&)
In case you've been observant, when defining the full_name method, we prefixed the "first_name" parameter with a & symbol commonly known as the Ampersand symbol. 
By prefixing the "first_name" parameter with an Ampersand(&) we let ruby know that a code block passed to the full_name method should be converted to a Proc object. Without it, we get an error below:
```ruby
3.2.6 :036 > def full_name(first_name)
3.2.6 :037 >   firstname = first_name.call
3.2.6 :038 >   full_name = firstname + " " + "Doe"
3.2.6 :039 > end
 => :full_name 
3.2.6 :040 > full_name do 
3.2.6 :041 >   "John"
3.2.6 :042 > end
(irb):36:in `full_name': wrong number of arguments (given 0, expected 1) (ArgumentError)
	from (irb):40:in `<main>'
	from /usr/share/rvm/gems/ruby-3.2.6/gems/irb-1.15.1/exe/irb:9:in `<top (required)>'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/irb:25:in `load'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/irb:25:in `<main>'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/ruby_executable_hooks:22:in `eval'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/ruby_executable_hooks:22:in `<main>'
3.2.6 :043 > 
```

Without the ampersand, Ruby no longer knows how to handle the code block passed to it and expects the full_name method to  be called like any other method in Ruby. And since we defined the full_name method to have one parameter, namely the first_name, Ruby expects that the method should be called along with one argument, which is exactly why we get the error above. In simple terms, Ruby is saying:

"*Hey, you defined the "full_name" method to take an argument but you've called the "full_name" method without providing an argument.*"

There is a way for Ruby to understand that a block passed to a given method should be handled as an explicit code block even though you've left out the Ampersand.

Consider the code below:
```ruby
def full_name(first_name)
  puts first_name.class
  firstname = first_name.call
  fullname = firstname + " " + "Doe"
end

full_name(Proc.new do
  "John"
end)

#OUTPUT
Proc
 => "John Doe"
```

In the above code, we see that the output is still the same as the initial "full_name" method regardless of the fact that we left out the Ampersand. The above code works because when passing the code block to the "full_name" method, we handled the conversion of the code block into a Proc object manually by calling Proc.new. 
In the output we also notice that the first_name object is a Proc object meaning the conversion of the code block into a Proc object was successful. 

This highlights the importance of the Ampersand. For a given code block to be termed as an explicit code block, ruby has to convert it into a Proc object. This can be achieved either automatically(by prefixing the "first_name" parameter with the Ampersand(&) and have Ruby handle the conversion of the code block into a Proc object behind the scenes as in the initial version of the full_name method) or manually where by we handle the conversion of the code block into a Proc object our selves by calling **Proc.new**. You can choose either based on what you prefer though my personal preference is to use the Ampersand as that's one less thing to think about in case things go wrong. 

When passing single line code blocks to methods for example the "full_name" method, you can use the syntax below:
```ruby
full_name(Proc.new{ "John" })
Proc
 => "John Doe" 
```

The syntax above is equivalent to the one below and returns the same output. 
```ruby
full_name(Proc.new do
  "John"
end)
Proc
 => "John Doe" 
```

And that marks the end of this post. In case you missed part 1 of the series, you can check it out [here](https://zhephyn.github.io/ruby/2025/04/16/an-introduction-to-ruby-code-blocks-part-1.html). 