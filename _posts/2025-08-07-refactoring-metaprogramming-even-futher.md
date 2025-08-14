---
layout: post
title:  "Refactoring metaprogramming even further"
date:   2025-08-07 23:53:47 +0300
categories: refactoring
---

```ruby
unless defined? VIEW_METHODS_MAPPING
  VIEW_METHODS_MAPPING = {
    index: [:index_fields, :display_fields],
    show: [:show_fields, :display_fields],
    edit: [:edit_fields, :form_fields],
    update: [:edit_fields, :form_fields],
    new: [:new_fields, :form_fields],
    create: [:new_fields, :form_fields]
    }
end
      
unless defined? VIEW_CARDS_MAPPING
  VIEW_CARDS_MAPPING = {
    index: [:index_cards, :display_cards],
    show: [:show_cards, :display_cards]
    edit: [:edit_cards, :form_cards],
    update: [:edit_cards, :form_cards],
    new: [:new_cards, :form_cards],
    create: [:new_cards, :form_cards]
    }
end

def fetch_fields
  
  possible_methods_for_view = VIEW_METHODS_MAPPING[view.to_sym]
  
  possible_methods_for_view&.each do |method_for_view|
    return send(method_for_view) if respond_to?(method_for_view)
  end

  fields
end

def fetch_cards
  possible_methods_for_view = VIEW_CARDS_MAPPING[view.to_sym]

  possible_methods_for_view&.each do |method_for_view|
    return send(method_for_view) if respond_to?(method_for_view)
  end
  
  cards
end
```
The code above applies a technique called metaprogramming to generate a myriad of methods ending in the "_fields" or "_cards" suffixes based on the current view. Take for example, the index view, the code generates 2 methods ending in the "_fields" suffix. These are; `index_fields` and `display_fields`. Using either of these methods, we can define which fields are displayed on the index view of a given resource. 

Take for example we have a post resource. If we want to display the title of the post along with its cover photo on the index page, we can achive this with the following code inside the post resource. 

```ruby
class Avo::Resources::Post < Avo::BaseResource

  def index_fields
   field :title, as: :text
   field :cover_photo, as: :file
  end

end
```
When we navigate to the index page of the Post resource, these two fields will be displayed. And for those unfarmiliar with Avo's DSL, what this means is that, we'll see both title and the cover photo displayed on the index page for each individual post. 

The same kind of thinking applies when thinking about the methods ending in the "_cards" suffix. As part of a card, we can define some extra information we'd want to display for a resource other than the fields. Take an example of the post resource from before, assuming we want to display the name of the creator of agiven post and the date on which a particular post was created, we can use a card for this. 

If we want this card to be displayed on the index page, the metaprogramming code above provides us with a method named `index_cards`. With this method, we can define what "information" is displayed as part of the card. Something like below:

```ruby
class Avo::Resources::Post < Avo::BaseResource
  def index_cards
    card Avo::Cards::ExtraPostInformation
  end
end
```
Then we'd define the information that get's displayed on the card. For this, we'd use a specific kind of card called a Partial card.

```ruby
class Avo::Cards::ExtraPostInformation < Avo::Cards::PartialCard
    # code to display name of post creator
    # code to display the date of creation of a post
end
```
With the above code, the card will be displayed on the Index page of the post resource with our defined information namely the Post creator's name and the date of creation of the Post.

Now that you're up to speed with what the code does, we'll proceed to refactoring it. In case you haven't noticed, the above code contains a duplication. 

Taking a look at the `VIEW_METHODS_MAPPING` and the `VIEW_CARDS_MAPPING`, we notice that these two mappings are almost identical, with the only difference being the fact suffix i.e. `_cards` and `_fields`. 

`VIEW_METHODS_MAPPING`:
```ruby
unless defined? VIEW_METHODS_MAPPING
  VIEW_METHODS_MAPPING = {
    index: [:index_fields, :display_fields],
    show: [:show_fields, :display_fields],
    edit: [:edit_fields, :form_fields],
    update: [:edit_fields, :form_fields],
    new: [:new_fields, :form_fields],
    create: [:new_fields, :form_fields]
    }
end
```

`VIEW_CARDS_MAPPING`:
```ruby
unless defined? VIEW_CARDS_MAPPING
  VIEW_CARDS_MAPPING = {
    index: [:index_cards, :display_cards],
    show: [:show_cards, :display_cards],
    edit: [:edit_cards, :form_cards],
    update: [:edit_cards, :form_cards],
    new: [:new_cards, :form_cards],
    create: [:new_cards, :form_cards]
    }
end
```

We could refactor these two mappings into a single mapping thereby doing away with the duplication.
Since the end goal of this metaprogramming functionality is to generate methods, opting for the name of the mapping to be `VIEW_METHODS_MAPPING` makes more sense compared to `VIEW_CARDS_MAPPING`. 

The refactored code for the mapping will look like below:

```ruby
unless defined? VIEW_METHODS_MAPPING
  VIEW_METHODS_MAPPING = {
    index: [:index, :display],
    show: [:show, :display],
    edit: [:edit, :form],
    update: [:edit, :form],
    new: [:new, :form],
    create: [:new, :form]
    }
end
```
By defaulting to one mapping for generating `fields` and `cards` methods, we've done away with the duplication in the initial code. We'll now proceed to make changes inside the `fetch_cards` and `fetch_fields` methods since this is where the code which generates the actual methods AKA the metaprogramming code is defined. 

First the `fetch_fields` method. 
We'll use the syntax `:"#{method_for_view}_fields"` to dynamically append the suffix when generating `fields` methods for a given view.

```ruby
# code ommitted for brevity
possible_methods_for_view = VIEW_METHODS_MAPPING[view.to_sym]

possible_methods_for_view&.each do |method_for_view|
  return send(:"#{method_for_view}_fields") if respond_to?(:"#{method_for_view}_fields")
end

fields
```
To explain this code a bit I'll pose a question? How does the above code generate the 2 methods namely `index_fields` and `display_fields`? Recall that these methods serve the purpose of defining the fields displayed on the index view of a resource. 

We start with the line of code below:
`possible_methods_for_view = VIEW_METHODS_MAPPING[view.to_sym]`:
- creates a variable named `possible_methods_for_view`.
- `view` corresponds to the current view, in which case this will be the index view. The `.to_sym` method turns the receiver(index) into a symbol eventually returning `:index`.

By this logic, we now have this `possible_methods_for_view = VIEW_METHOD_MAPPING[:index]`. Since `VIEW_METHODS_MAPPING` is a hash with one of the keys inside this hash being the `:index` symbol, as seen below:
`index: [:index, :display]`, 
we are able to access the 2 symbol values corresponding to the `:index` key namely `:index` and `:display`. We'll use these as the first parts of the index view method names. 

The metaprogramming code below:  

`return send(:"#{method_for_view}_fields") if respond_to?(:"#{method_for_view}_fields")`

will append the `_fields` suffix to the 2 values namely `index` and `display` such that we end up with 2 methods for the index view namely: `index_fields` and `display_fields`.

For these 2 generated methods, we loop through them to see if the view specific `index_fields` method was defined in the resource file, and if this evaluates to true, the method returns. If it evaluates to false, we proceed to check whether the less-specific `display_fields` method was defined, and if yes the method returns. The key takeaway here, is that we check the generated methods based on what was defined inside the resource file, with the view-specific methods such as `index_fields` taking precedence over context-specific methods such as `display_fields`. At any one point during the check, if we find a match, we return and the method exits, otherwise we continue the check with the next available method for that view.

`display_fields` is referred to as a context-specific method because it not only applies to the index view but also the show view which are both display views. 

By using this syntax `:"#{method_for_view}_fields"` instead of `method_for_view`, we have the code automatically append `_fields` when generating a `fields` method thereby removing the initial duplication in the mapping. 

> Both the initial and the new metaprogramming implementations utilize 2 methods named `send` and `respond_to?`.
> These are covered in more detail in the posts linked at the end of this post. 

For the `fetch_cards` method, we'll use a variation of the same syntax just like in the `fetch_fields` method, to have the `_cards` suffix dynamically appended when generating `cards` methods for a specific view. 

`:"#{method_for_view}_cards"`

```ruby
def fetch_cards
  possible_methods_for_view = VIEW_METHODS_MAPPING[view.to_sym]

  possible_methods_for_view&.each do |method_for_view|
    return send(:"#{method_for_view}_cards") if respond_to?(:"#{method_for_view}_cards")
  end

  cards
end
```
Because we specify the suffix to be `_cards` instead of `_fields` in this line of code, `:"#{method_for_view}_cards"`, we'll end up with 2 cards methods generated for the index view namely `index_cards` and `display_cards`. 

The new version of the initial code will look like below:

```ruby
unless defined? VIEW_METHODS_MAPPING
  VIEW_METHODS_MAPPING = {
    index: [:index, :display],
    show: [:show, :display],
    edit: [:edit, :form],
    update: [:edit, :form],
    new: [:new, :form],
    create: [:new, :form]
  }
end

def fetch_fields
  possible_methods_for_view = VIEW_METHODS_MAPPING[view.to_sym]
  
  possible_methods_for_view&.each do |method_for_view|
    return send(:"#{method_for_view}_fields") if respond_to?(:"#{method_for_view}_fields")
  end

  fields
end

def fetch_cards
  possible_methods_for_view = VIEW_METHODS_MAPPING[view.to_sym]

  possible_methods_for_view&.each do |method_for_view|
    return send(:"#{method_for_view}_cards") if respond_to?(:"#{method_for_view}_cards")
  end

  cards
end
```

