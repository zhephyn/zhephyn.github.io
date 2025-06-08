---
layout: post
title:  "An Introduction To Ruby Code blocks part 1"
date:   2025-04-16 10:30:20 +0300
categories: ruby
---

## My First Ruby Problem

When i had started out to learn ruby, one of the earliest problems i was presented with was related to arrays. In this problem, you have an array. Then you have to iterate over that array such that you have each element of the array printed on its own line in the interpreter. For visual purposes, here is the problem plus its solution below implemented in irb.

```ruby
numbers = [1,2,3]
numbers.each do |number|
  puts number
end
```

The output:
```bash
1
2
3
 => [1, 2, 3]
```
The solution is pretty simple, you utilize the "each" method for the iteration. This method is available for all arrays as part of the Array class. In addition to this, the solution utilizes a do-end block which specifies what should be done for each element in the array. In our case, we just print each element on its line in the irb console, but there are endless possibilities regarding what you could do with an element you've iterated over. 

## Enter Code Blocks

Unbeknownst to me back then, that do-end block has a specific term ruby programmers used to refer to it and it can do way more, than just printing out array elements to the console.

The name of that do-end block is a code block, and to be specific about the kind of code block, since code blocks come in 2 variants, the code block in the array example above is an implicit code block. We'll go over why it's called an Implicit code block later on. But first, a short introduction to Ruby code blocks.

Put simply a code block is an executable piece of code that tends to be associated with a given method. This method is responsible for executing the code block passed to it.

### Your First Ruby Code Block

Let's say you have a method below:

```ruby
def my_method
  yield if block_given?
end
```

Then at any other place in your program, you have the code below:
```ruby
my_method do
  puts "This is the output   derived from executing the code block"
end
```

The above can also be written as:
```ruby
my_method { puts "This is the output derived from executing the code block"}
```

When we run either of the examples above in the ruby interpreter, we'll have the statement below as the output:

```ruby
"This is the output derived from executing the code block"
```

### Behind the scenes

So how did we arrive at this output:

First and foremost, let’s examine the method. We defined a method named "my_method". Inside the method, we have a yield keyword bound to an if statement. 
```ruby 
yield if block_given?
```
Inside the method, we perform a conditional check. We evaluate whether a code block was passed to the “my_method” method using the block_given? method which evaluates to true or false. If the conditional check evaluates to true i.e a code block was passed when calling the method, we proceed to return the output of executing the code inside the code block. To return this output for the user to see in the interpreter, we use the yield keyword.

All in all 2 things are crucial within this method for the desired functionality to be achieved:
- The block_given? Method which evaluates to a Boolean and performs a conditional check.
- yield which returns the output of executing the code inside the code block

We can also write the same method as:
```ruby
def my_method
  if block_given?
    yield
  end
end
```


Now onto the code block itself.
This is responsible for the output we see in the interpreter. 
So in the code above , what we are doing is, we call a method and pass a code-block to it. And since the method does a conditional check with the block_given? method to see if a code block was passed to the method when it was called, the “my_method” method knows exactly how to handle this situation.

In the example above, so many moving parts are responsible for the output we end up seeing in the irb session. 
Now a hypothetical, what would be the output of running this program if we don’t perform the conditional check?

In case we defined the same method without the conditional check like below, then proceed to pass a code block to that method, we observe that no output is returned in the irb session. 
```ruby
def my_method
end
```

```ruby
my_method do
  puts "This is the output derived from executing the code block"
end
```

Trying to run the above will result in "nil" as the output. This is because, as much as our syntax for passing a code block to my_method is correct, the method itself doesn't know what to do with the code block, which is why we don't get the expected output.
The "nil" output wasn't returned by the my_method, its pretty standard output.
So to be more specific about what happened, when you pass a code block to a method that isn't in the know about what it should do with the code block and proceed to call that method, nothing is executed and nothing is returned.

### Why the name "Implicit code block"?

Take the example method below:
```ruby
def say_name(name)
  puts "Your name is #{name}"
end
```

To have this method return a name of the current user, we simply call the method and pass in an argument of whatever name we want to return.
```ruby
say_name("Newton")
```
Output:
```ruby
Your name is Newton
```

Contrast the method above with the “my_method” method.

The method “my_method” differs from the “say_name” method in 1 major way:
When defining the “my_method” method, we didn’t specify any parameters for it. Because of this, when the method is called(along with the code block passed to it), the code block is not passed as an argument to the method. Since we call the method and pass a code block which isn’t treated by the method as an argument, this code block is referred to as an implicit code block.

This poses a few implications:
Because the code block is passed to the method not as an argument, what we can achieve with this code block is limited. 

Think about it, if we were able to pass the code block as an argument when calling the method, we would be able to associate this code block with a parameter which means we can store whatever is in the code block and therefore use it later instead of instantly. This specific kind of code block is an Explicit code block which we'll look at in part 2 of this section.

## A Closer look at Ruby's "each" method.

Like we mentioned earlier, the solution for the array problem used the "each" ruby method whose underlying operating principle is similar to that of Implicit code blocks in Ruby. We can confirm this by taking a look at the source of Ruby's "each" method. 
```C
VALUE
rb_ary_each(VALUE ary)
{
  long i;
  ary_verify(ary);
  RETURN_SIZED_ENUMERATOR(ary, 0, 0, ary_enum_length);
  for (i=0; i<RARRAY_LEN(ary); i++) {
    rb_yield(RARRAY_AREF(ary, i));
  }
  return ary;
}
```
In this code, a couple things are happening but a few stand out.  

1. The "each" method utilizes an equivalent of "yield" named "**rb_yield**".
2. **RETURN_SIZED_ENUMERATOR**  checks for the presence of a block similar like what the block_given? method employed by the method "my_method" does. 
See the code attached below:

```C
#define RETURN_SIZED_ENUMERATOR(obj, argc, argv, size_fn) do { \
  if (!rb_block_given_p()) \
    return SIZED_ENUMERATOR(obj, argc, argv, size_fn); 
} while (0)
```

In the above code, we have a function named **rb_block_given_p()** whose purpose is to check whether the current method, in this case the "each" method is given a block. The rb_block_given_p() function as well as the block_given? ruby method, are of a similar importance, that is to check whether a given method was passed a block. 

Based on the above findings, we can confidently say that the "each" ruby method operates based on the principle of implicit code blocks under the hood. 

## Conclusion
In case you're interested, you read more about RETURN_SIZED_ENUMERATOR, ruby's each method as well as the rb_block_given_p() by visiting the resources below:
1. [Each Ruby method](https://ruby-doc.org/3.4.1/Array.html#method-i-each)
2. [RETURN_SIZED_ENUMERATOR macros](https://docs.ruby-lang.org/capi/en/master/dc/d1b/include_2ruby_2internal_2intern_2enumerator_8h_source.html)
3. [rb_block_given_p function](https://docs.ruby-lang.org/capi/en/master/d7/d19/group__defmethod.html)

This part served as an introduction to implicit code blocks, and looked at the use cases for implicit code blocks which is iterating over elements in an array. In part 2, we'll look at yet another type of Ruby code blocks named Explicit code blocks and highlight how this specific kind of code block solves the short comings of Implicit code blocks. 