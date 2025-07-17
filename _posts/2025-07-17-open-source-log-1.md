---
layout: post
title:  "Opensource log 1"
date:   2025-07-17 23:53:47 +0300
categories: ruby
---
This is part 1 of my series documenting one new thing I learned for every open source PR I've worked on.

## Back story
So for this issue, the idea was simple. Add a bunch of view specific formatters such that a value displayed on a given view page is formatted
based on that view we're currently on. For each of the views namely new, show, edit and index, each of these had its respective formatters. 

new: format_new_using
edit: format_edit_using
index: format_index_using
show: format_show_using
show & index: format_display_using(similar to format_show_using and format_index_using)

Each of the above formatters takes the value we want to format and "processes" it in such a way that we display the raw value as its saved in the database for the edit page and then display the "formatted value" on the index and show pages. 

Therefore, If we have the value as the number 10 and we want to this value is supposed to be displayed as a dollar amount, therefore, instead of 
10, we would want the user to show 10.00 instead of 10 on the index and show pages. To achieve this, we'd have to do this:
```ruby
field :amount, as: number, format_index_using: -> { number_to_currency value }
```
With this in mind, based on each of the views, you can clearly see that we wanted to implement over 5 view formatters. 

## Initial Approach
Given that we want to format the value, we need to look through the code base and find the method responsible for returning the value displayed on each of the view pages, keeping in mind that this method is the one responsible for returning the display value for all fields.

In the above field code sinppet, we can see that its related to the number field. However, we don't want the formatters to be restricted to just the number field. We want them to be available for all fields in our application such that the code below works:
```ruby
field :amount, as: :code, format_index_using: -> { #method used to format the code displayed, value }
```
By this logic, we know that this method is probably defined in the base field at this file path ```lib/avo/fields/base_field.rb```. Therefore, since all fields inherit from the base field, making the changes here will allow the all the fields that inherit from the base field to have this functionality available to them through inheritance.

```ruby
def value(property = nil)
    return @value if @value.present?
    property ||= @for_attribute || @id
    # Get record value
    final_value = @record.send(property) if is_model?(@record) && @record.respond_to?(property)
    # On new views and actions modals we need to prefill the fields with the default value if value is nil
    if final_value.nil? && should_fill_with_default_value? && @default.present?
        final_value = computed_default_value
    end
    # Run computable callback block if present
    if computable && @block.present?
        final_value = execute_context(@block)
    end
    # Run the value through resolver if present
    if @format_using.present?
        final_value = execute_context(@format_using, value: final_value)
    end
    if @decorate.present? && @view.display?
        final_value = execute_context(@decorate, value: final_value)
    end
    final_value
end
```
So inside this file, we'll make our changes. These formatters will be implemented similar to how ```format_using``` is implemented. 
So inside this method, we'll add a conditional check which follows the pattern below for each of the formatters. 

An implementation of ```format_index_using```:

```ruby
if @format_index_using.present? && @view.index?
    final_value = execute_context(@format_index_using, value: final_value)
end
```
What this code does is basically this: 
```@view``` is an avo specific "#add appropriate word here" which helps you check that the type of current page. If we are on the index page, this will return true. 

For this to work, 2 conditions should be satisfied. One, we check that the ```format_index_using``` formatter is present, then we check whether we are on the index page, since this is an index page specific formatter. Both these conditions must evaluate to true. 
From our definition of the field we passed in a lambda:
```ruby
field :amount, as: number, format_index_using: -> { number_to_currency value }
```
The code above is similar to the one below with parentheses:
```ruby
field :amount, as: number, format_index_using: -> { number_to_currency(value) }
```
```number_to_currency``` is a rails number helper method which formats a value like below and in this case, we've passed it the value as the argument.

```ruby
number_to_currency("12x34")  # => $12x34 change this to use a number like 10
```
What the method does, is it changes a number passed to it for example 10 such that its displayed in the form of dollar currency. Therefore when we do

```ruby
number_to_currency(10) # => $10.00
```
Remember, we passed this entire code as a lambda and therefore, we'll need a way to execute the lambda. That is exactly what ```execute_context``` does.

The output of the lambda execution is what is assigned to the variable ```final_value```. 

Inside the ```value``` method, to be able to access(read and write) the ```format_index_using``` formatter as an instance variable like below:
```ruby
@format_index_using.present?
```
We'll have to make some changes to the ```initialize``` method as well as make the attribute readable by using ```attr_reader```.

```ruby
 #where the other attr_reader are defined, add one for format_index_using
attr_reader :format_index_using
```
then in the ```initialize``` method:
```ruby
@format_index_using = args[:format_index_using]
```
Upon doing that, you can try using the newly added ```format_index_using``` formatter and as you can see it works as expected. 

Now as you've probably guessed, to implement the rest of the view formatters, we'll have to make the above changes for each of them, just like we did for ```format_index_using```. 

The final result is a bunch of attr_reader's that look like below:
```ruby
attr_reader :format_display_using
attr_reader :format_index_using
attr_reader :format_show_using
attr_reader :format_edit_using
attr_reader :format_new_using
attr_reader :format_form_using
```

An ```initialize``` method that looks like below:
```ruby
@format_display_using = args[:format_display_using]
@format_index_using = args[:format_index_using]
@format_show_using = args[:format_show_using]
@format_edit_using = args[:format_edit_using]
@format_new_using = args[:format_new_using]
@format_form_using = args[:format_form_using]
```

And finally a ```value``` method which looks partly like this:
```ruby
# upper code ommitted for brevity
if @format_using.present?
    final_value = execute_context(@format_using, value: final_value)
end
if @format_display_using.present? && @view.display?
    final_value = execute_context(@format_display_using, value: final_value)
end
if @format_index_using.present? && @view.index?
    final_value = execute_context(@format_index_using, value: final_value)
elsif @format_display_using.present? && @view.display?
    final_value = execute_context(@format_display_using, value: final_value)
elsif @format_using.present?
    final_value = execute_context(@format_using, value: final_value)
end
if @format_show_using.present? && @view.show?
    final_value = execute_context(@format_show_using, value: final_value)
 elsif @format_display_using.present? && @view.display?
          final_value = execute_context(@format_display_using, value: final_value)
elsif @format_using.present?
          final_value = execute_context(@format_using, value: final_value)
end

if @format_edit_using.present? && @view.edit?
    final_value = execute_context(@format_edit_using, value: final_value)
elsif @format_using.present?
    final_value = execute_context(@format_using, value: final_value)
end

if @format_new_using.present? && @view.new?
    final_value = execute_context(@format_new_using, value: final_value)
elsif @format_using.present?
    final_value = execute_context(@format_using, value: final_value)
end

if @format_form_using.present? && @view.new?
    final_value = execute_context(@format_form_using, value: final_value)
elsif @format_using.present?
    final_value = execute_context(@format_using, value: final_value)
end
if @decorate.present? && @view.display?
    final_value = execute_context(@decorate, value: final_value)
end
final_value
end
```
As you can see, for every formatter, we use the ```format_using``` formatter as a fallback. For example, when implementing the ```format_form_using``` formatter, we check whether the formatter has been defined and that we are on the new page, If both of these evaluate to true, we proceed to format the value using ```format_form_using``` otherwise, we format using ```format_using```. By this logic, ```format_using``` is available for all views regardless of the type. The same kind of logic applies when implementing the ```format_new_using``` formatter. We check if the view specific formatter was defined, if yes, we use that formatter. If not, we use ```format_using``` which is available for all views. We end up having and if-elsif block. 

```ruby
if @format_form_using.present? && @view.new?
    final_value = execute_context(@format_form_using, value: final_value)
elsif @format_using.present?
    final_value = execute_context(@format_using, value: final_value)
end
```
For the implementation of the ```format_show_using``` and ```format_index_using``` formatters, still we want to check if a view specific formatter is available i.e ```format_show_using``` for the show page and ```format_index_using``` for the index page. If no view specific formatter is defined, we default to the ```format_using``` formatter available for all views.  In addition to this, for both these views, we check for whether the ```format_display_using``` formatter was defined. This formatter allows us to define that the value should be formatted in the same way for both the index and show pages which would otherwise require us to define both of them at once.
So, instead of this:
```ruby
field :amount, as: number, format_index_using: -> { number_to_currency(value) }
field :amount, as: number, format_show_using: -> { number_to_currency(value) }
```
We do this:
```ruby
field :amount, as: number, format_display_using: -> { number_to_currency(value) }
```
All in all, for both the ```format_index_using``` and ```format_show_using``` we end up having an if-elsif-else block. 

## Problems with the if-statement approach. 
The multiple-if statement approach is not faulty, it actually works. The only problem is that we can see the method becomes very lengthy very quickly and in a case where we would want to add future formatters, the method would get even lengthier. 
For projects that implement a CI check which limits methods to not exceed a certain number of lines, you'll see that this method, while perfectly functional will be flagged and the CI will end up failing. 

## Enter metaprogramming
This is one of the hardest things to wrap your head around when you're starting out in ruby. Things can get confusing really fast. 
The simplest explanation of metaprogramming is that "we have code that works but is kind invisible-ish". Of course, the code is there and by "invisible-ish", I don't mean this in the literal sense of the word. 

attach a better explanation from meta-programming ruby

Any way, so how would we refactor our multiple if statements with meta-programming?

We can refactor this code into the private method below:
```ruby
def format_value(value)
        final_value = value

        formatters_by_view = {
          index: [:format_index_using, :format_display_using, :format_using],
          show: [:format_show_using, :format_display_using, :format_using],
          edit: [:format_edit_using, :format_using],
          new: [:format_new_using, :format_form_using, :format_using],
        }
        current_view = @view.to_sym
        applicable_formatters = formatters_by_view[current_view]

        if applicable_formatters
            applicable_formatters.each do |formatter|
                formatter_value = instance_variable_get("@#{formatter}")
                if formatter_value.present?
                    final_value = execute_context(formatter_value, value: final_value)
                    return
                end
            end
        end
        final_value
end
```
then inside the ```value``` method where we had the multiple if-statements we can do this:
```ruby
# Format value based on available formatter
final_value = format_value(final_value)
```
Explanation of how the code works:
Take an example of if we are currently on the new page:
we define applicable formatters for each view;
we determine what view we are currently on, if we are on the new page:
```current_view = @view.to_sym```
will return ```:new```
Then we store the applicable formatters for the ```new``` page inside a hash which is assigned to the variable applicable_formatters. 
Run through applicable_formatters with the each method, for each of the formatters, return an instance_variable for example ```@format_new_using, @format_form_using, @format_using```
check is the formatter you returned is present, 
From first to last, loops through ```@format_new_using, @format_form_using, @format_using```
@format_new_using.present? returns "true"
if yes, execute the lambda and get out of the if block and return the final value. If not, proceed to the next formatter(@format_form_using.present? returns "true"). If its still not present, use the default formatter(@format_using.present? returns "true") named ```format_using```
Therefore, if we defined:
```ruby
field :amount, as: :number, format_new_using: -> {number_to_currency value}
```
in this below
```ruby
new: [:format_new_using, :format_form_using, :format_using]
```
the first formatter we'll check for is @format_new_using and since its the one which was defined, the program will stop here and format the value based on this formatter. The rest of the formatters are not "checked"

If we did this:
```ruby
field :amount, as: :number, format_form_using: -> {number_to_currency value}
```
The first formatter(@format_new_using) will be checked, but since it wasn't defined, it will be skipped and we won't use it for formatting. We'll then move on to the next formatter in line which is @format_form_using and since this was the defined formatter, we'll use this to format the value. In this case, the program won't proceed further to format_using as it already found the formatter which was defined. In a case, where format_using was the defined formatter, we run through format_new_using and then format_form_using until we get to format_using which was the defined formatter and we use this for formatting. 

And that's it. By using metaprogramming, we made our implementation of the view formatters less cluttered and most importantly, maintainable. 