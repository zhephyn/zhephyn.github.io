---
layout: post
title:  "Learn about Ruby's send method with a practical example"
date:   2025-08-05 23:53:47 +0300
categories: refactoring ruby
---
The `send` method is one of the most commonly used Ruby methods when implementing metaprogramming solutions to problems in Ruby. For this reason, it is imperative that you understand how it works and the context in which its a method of choice. There is no better way to do this than with a practical example. 

Keep the code below in mind as everything that follows in this post will refer to it.

You have a `document.rb` file. This file will contain the metaprogramming code which serves the sole purpose of dynamically building a set of methods ending in the `_attributes` suffix with one example being the `index_attributes` method in the Post class.

```ruby
class Document
  attr_accessor :view 

  VIEW_ATTRIBUTES_MAPPING = {
    index: [:index_attributes, :display_attributes],
    show: [:show_attributes, :display_attributes],
    edit: [:edit_attributes, :form_attributes],
    new: [:new_attributes, :form_attributes],
    create: [:new_attributes, :form_attributes],
    update: [:edit_attributes, :form_attributes]
   }

  def fetch_attributes
    possible_attributes_for_view = VIEW_ATTRIBUTES_MAPPING[view.to_sym]
    
    possible_attributes_for_view&.each do |method_for_view|

      return send(method_for_view) if respond_to?(method_for_view)
    end
  end
end
```

Then a `post.rb` file which inherits from what is defined in `document.rb`. In this file, we determine a method named `index_attributes` which returns an array of items when called. 

```ruby
class Post < Document
  attr_accessor :view

  def initialize(view)
    @view = view
  end

  def index_attributes
    [title, body, cover_photo, creator]
  end
end
```

At some point in the base.rb file, we have a call to the `send` method. By examining this line of code, we'll be able to answer questions about how the `send` method works. 
1. What is the receiver object when calling the `send` method in this line of code?
2. How does the class of this receiver object change in various contexts?
3. What purpose does the `send` method perform in this code and why is it referred to as a metaprogramming method?
4. The `send` method in this code is used in conjunction with the `respond_to?` method. Why is this?

## Send has been summoned but by who?
Usually a Ruby method is called like below:
```ruby
receiver.method
```
The `receiver` being the object which the `method` is called on. However, the line of code with the `send` method looks peculiar in the sense that, we see a method named `send` being called but by who? Who is calling the `send` method? The line with the `send` method call doesn't specify an explicit receiver for the method call.
`return send(method_for_view) if respond_to?(method_for_view)`

If you're an experienced Ruby developer, its probably quick for you to know that the above line of code is synonymous with the line of code below with the `self` being usually ommitted for stylistic reasons.
`return self.send(method_for_view) if self.respond_to?(method_for_view)`.

Now that we've established that `self` is the receiver for the `send` method call, all is starting to make sense. However, we are still left with one question to answer. What does `self` refer to in this context?

Let's refresh our minds by going back to what this line of code does. The line is supposed to check if a bunch of methods ending in the `_attributes` suffix are defined some where, possibly a Ruby class of some sort. If these methods are defined in that specific place i.e `respond_to?` returns true, the defined methods ending in the `_attributes` suffix will be called on an object created by that specific class where the method was defined in the first place. 

In the `post.rb` file, we have an `index_attributes` method. If we replace `method_for_view` with `index_attributes`, the line of code with the send method call will be equivalent to `return self.send(:index_attributes) if self.respond_to?(:index_attributes)`.

In English, the line of code above loosely translates to "Call the method named `index_attributes` on an object created by a class where the `index_attributes` method is defined but, first perform a check that the `index_attributes` method is actually defined in this class. Only if its defined, then call the method."

This highlights two things:
1. The call to `respond_to?` happens first. This checks if the `index_attributes` is defined in the class. Therefore, its safe to say that the value of `self` in `self.respond_to?(:index_attributes)` is an instance object whose mother class is the class where the `index_attributes` method is defined. 

2. The call to `send` happens last. This actually calls the "already defined" method on the specific instance object. Therefore, even in this case `self` is equivalent to an instance object created from the class where the `index_attributes` method is defined. 

In both cases, since the `index_attributes` method is defined in the Post class, the object which is `self` will translate to an instance object of the Post class. By this logic, we expect the method to return an array of the following items; `[title, body, cover_photo, creator]`. Let's try this out in an irb session.

Temporarily modify the `fetch_attributes` method to point out the current value of `self`.

```ruby
def fetch_attributes
  possible_attributes_for_view = VIEW_ATTRIBUTES_MAPPING[view.to_sym]
   
  possible_attributes_for_view&.each do |method_for_view|
    puts "Current value of self is: #{self}"
    return self.send(method_for_view) if self.respond_to?(method_for_view)
  end
end

```

Create a new post instance and proceed to call the `fetch_attributes` method on it.

```ruby
3.2.6 :146 > new_post = Post.new(:index)
 => #<Post:0x0000776499e39bb0 @view=:index> 
3.2.6 :147 > new_post.fetch_attributes
The current value of self is: #<Post:0x0000776499e39bb0>
 => [:title, :body, :cover_photo, :creator] 
3.2.6 :148 > 
```
As you can see, the current value of `self` is the newly created instance object named post.

Also, when we call `post.fetch_attributes`, we get the array of items returned to us. This is because since we has set the current view to be index, 2 methods related to the index view will be created namely the `index_attributes` and `display_attributes`. The line `return send(method_for_view) if respond_to?(method_for_view)` will check for the `index_attributes` method first and if it was defined inside the Post class. Since this is the case, the method will return the output of calling the `index_attributes` method ([:title, :body, :cover_photo, :creator] ) and thereafter return.

Now that we've established who summoned the `send` method, we'll proceed to investigate the changing nature of self.

## Any object can be self depending on context
Inside the Post class, if you look closely, you'll notice a subtle detail. While `send(method_for_view)` can be written as `self.send(method_for_view)`, the method in the Post class is not written as below.

```ruby
class Post < Document
  def self.index_attributes
    [title, body, cover_photo, creator]
  end
end
```
Why is this the case? To figure this out, lets create a new instance of the Post class and proceed to call the `index_attributes` method on it. This results into an error. Why is this? 

```ruby
3.2.6 :222 > post = Post.new(:index)
 => #<Post:0x0000776499f7bb68 @view=:index> 
3.2.6 :223 > post.index_attributes
(irb):223:in `<main>': undefined method `index_attributes' for #<Post:0x0000776499f7bb68 @view=:index> (NoMethodError)

post.index_attributes
    ^^^^^^^^^^^^^^^^^
	from /usr/share/rvm/gems/ruby-3.2.6/gems/irb-1.15.2/exe/irb:9:in `<top (required)>'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/irb:25:in `load'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/irb:25:in `<main>'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/ruby_executable_hooks:22:in `eval'
	from /usr/share/rvm/gems/ruby-3.2.6/bin/ruby_executable_hooks:22:in `<main>'
3.2.6 :224 > 
```
This is because defining the `index_attributes` method as `self.index_attributes` changes it from being an instance method to a class method. What this means is that we can nolonger call it on instances of the Post class but rather only on the Class itself. As you can see below, calling the method on the class itself doesn't throw an error. 

Also, its important to keep in mind that when the metaprogramming code is generating methods ending in the `_attributes`, the methods for example the `index_attributes` method will be generated as `index_attributes` instead of `self.index_attributes`. 

With this, we can conclude, `self` can stand for anything. From the class itself to an instance of a class. 

## Is send really that important? Where would we be without send?
Without the `send` method, the `fetch_attributes` would potentially look like below.
```ruby
def fetch_attributes
    return index_attributes if respond_to?(index_attributes)
    return display_attributes if respond_to?(display_attributes)
    return show_attributes if respond_to?(show_attributes)
    return edit_attributes if respond_to?(edit_attributes)
    return form_attributes if respond_to?(form_attributes)
    return new_attributes if respond_to?(new_attributes)
end
```
As you can see, we'd have to manually write return statements to check if each and everyone of the possible methods is defined.This gets tiring pretty fast, especially if you're dealing with a lot of methods. With send in our toolbox, we can dynamically generate and call the currently defined method(s) on whatever self is. Put simply, `send` allows us to write maintainable code while being lazy along the way.

## My name is send and I wanna tell you about my colleague respond_to?
The `respond_to?` method is what checks if a method is defined in the class. Without it, the output below is what we'd run into. 

## A subtle requirement for using send method
Let's modify the code a bit to identify the value of `method_for_view`.

```ruby
  def fetch_attributes
    possible_attributes_for_view = VIEW_ATTRIBUTES_MAPPING[view.to_sym]
    
    possible_attributes_for_view&.each do |method_for_view|
      puts "Value of method_for_view is: #{method_for_view}"
      puts " Class of method_for_view: #{ method_for_view.class}"
      return send(method_for_view) if respond_to?(method_for_view)
    end
  end
```

As you can see, `method_for_view` is a Symbol so, inside the parentheses, when calling the `send` method, we pass in a symbol and not a method name.

```ruby
3.2.6 :314 > post.fetch_attributes
Value of method_for_view is: index_attributes
Class of method_for_view is: Symbol
Value of method_for_view is: display_attributes
Class of method_for_view is: Symbol
 => [:index_attributes, :display_attributes] 
3.2.6 :315 > 
```

This reveals a subtle detail about the `send` method in Ruby which is that allows 2 options for passing an already defined method to send inside parentheses with one of them being as a symbol(like the current code) and the other as a string. 

And that's pretty much all I can say about the `send` method in ruby. Before I depart however, I'll leave you with a question. What happens incase we pass a non-existent method to the `send` method? Try it out in the browser and see the result for yourself. 