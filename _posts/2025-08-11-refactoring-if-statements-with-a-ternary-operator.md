---
layout: post
title:  "Refactoring If statements with a ternary operator"
date:   2025-08-11 23:53:47 +0300
categories: refactoring
---

Junior ruby developers have this very strong compulsion to think of solving coding problems in terms of if statements. And before you come at me, I came to this realization simply because I'm a junior myself. This isn't me trashing if statements as a problem solving approach. On the contrary, if statements are sometimes absolutely necessary and if you were to take a deep dive of some of the best rails/ruby code bases out there, you'll notice that they employ if statements in multiple places. 

Take for example, you have a single condition and you have two desired outputs. One output is returned when the condition is true and the other is returned if the condition is false. It wouldn't make sense for you to apply a complex technique for example like metaprogramming to implement this. A simple if statement would suffice.

So we've established two things, one is that if statements are important. The second is that there are cases when a simple if statement will be the most appropriate approach for the problem you're solving. Following these two realisations, in this post, we'll look at one of the approaches of refactoring your if statements such that they look more "beautiful". For those that have contributed to open source projects before, without even realizing, you might've been required to apply this technique to rewrite your if statements before your PR was merged. If that was you, this post will get you up to speed with what you probably did. So let's dive right in. 

A ternary operator is a common approach to refactoring if statements in your Ruby project. 

Just like the if-statement its meant to refactor, the ternary operator consists of 3 parts namely:

- A condition
- what should be returned if the condition evaluates to true
- what should be returned if the condition evaluates to false

Having the components of the ternary operator in mind, we can visualize it to look like below:

``` condition ? return this if condition is true : return this if condition is false ```

Now that we know the structure of a ternary operator, let's proceed to use it to refactor an if statement in actual code. 

Consider the method below:

```ruby
def value
  stored_value = super

  if @display_code
    stored_value
  else
    countries[stored_value]
  end
end
```
Refactored using a ternary operator, the if statement in the above method would look like below:

``` @display_code ? stored_value : countries[stored_value] ```

Therefore the final method would look like this:

```ruby
def value
  stored_value = super 

  @display_code ? stored_value : countries[stored_value] 
end
```
With the ternary operator, we managed to refactor the if statement in the method to be more cleaner and maintainable. 

I applied this technique as part of a contribution I made into Avo and in case you are interested in learning more about the context of this PR, consider checking it out here. 

https://github.com/avo-hq/avo/pull/4005
