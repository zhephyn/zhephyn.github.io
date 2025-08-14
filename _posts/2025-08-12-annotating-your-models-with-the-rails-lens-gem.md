---
layout: post
title: Annotating models in your rails app with the rails-lens gem
date: 2025-08-12 23:10:12 +0300
category: tutorial
---
At some point in a Rails project, you'll have to go through a period of understanding the models and the relationships inside them. This will be more likely before you start contributing to a rails open source project or at your new job. This part of onboarding usually involves a dauting process of going back and forth between the model files and the `schema.rb` file with the sole purpose of mapping relationships between database tables as well as establishing what the models them selves are capable of. Luckily for us, there is a better way to achieve the same, and with less frustration by using the `rails-lens` gem. 

Rails Lens will annotate model files in your app and highlight relationships between database tables in your application, all by running a single command. For purposes of this tutorial, I've set up a sample Rails app.

This rails app is the usual rails blog application. It has a couple models namely Post, User, Comment and Category. These models have various associations between each other in the sense that, a post belongs to a user, has many comments and belongs to a category. Attached below are the model files.

```ruby
```

To set up rails lens in the application, add the line below to your gem file. I've chosen to install version 
`0.2.6`

```ruby
```

Run `bundle install`

If the installation is successful, you'll see the output below in your terminal when you run `bundle info rails_lens`

```ruby
  * rails_lens (0.2.6)
	Summary: Comprehensive Rails application visualization and annotation
	Homepage: https://github.com/seuros/rails_lens
	Source Code: https://github.com/seuros/rails_lens
	Changelog: https://github.com/seuros/rails_lens/blob/main/CHANGELOG.md
	Path: /usr/share/rvm/gems/ruby-3.3.4/gems/rails_lens-0.2.6
```

Note: At the time of writing this article, you have to specify the version of rails-lens you want to be installed. By just adding `gem 'rails-lens'` to your gem file and running `bundle install`, the version that ends up being installed is a placeholder version named `0.0.0`. With this version, you'll run into the error below when you run the command responsible for annotation.

```ruby
bundler: command not found: rails_lens
Install missing gem executables with `bundle install`
```

After installing the `rails-lens` gem, now onto the fun part, annotation.

To annotate all the model files in the application, run the command below:

```ruby
bundle exec rails_lens annotate
```
The above command adds annotations the 4 model files in our rails app. 

Let's take a quick look at the `post.rb` model file.

```ruby
# <rails-lens:schema:begin>
# table = "posts"
# database_dialect = "SQLite"
#
# columns = [
#   { name = "id", type = "integer", primary_key = true, nullable = false },
#   { name = "title", type = "text", nullable = true },
#   { name = "body", type = "text", nullable = true },
#   { name = "category_id", type = "integer", nullable = false },
#   { name = "user_id", type = "integer", nullable = false },
#   { name = "created_at", type = "datetime", nullable = false },
#   { name = "updated_at", type = "datetime", nullable = false }
# ]
#
# indexes = [
#   { name = "index_posts_on_user_id", columns = ["user_id"] },
#   { name = "index_posts_on_category_id", columns = ["category_id"] }
# ]
#
# foreign_keys = [
#   { column = "user_id", references_table = "users", references_column = "id" },
#   { column = "category_id", references_table = "categories", references_column = "id" }
# ]
#
# == Notes
# - Association 'comments' should specify inverse_of
# - Association 'comments' has N+1 query risk. Consider using includes/preload
# - Column 'title' should probably have NOT NULL constraint
# - Column 'body' should probably have NOT NULL constraint
# - Large text column 'title' is frequently queried - consider separate storage
# - Large text column 'body' is frequently queried - consider separate storage
# <rails-lens:schema:end>


class Post < ApplicationRecord
  has_many :comments, dependent: :destroy
  belongs_to :category
  belongs_to :user
end
```
With the lines below, we can identify that this is a schema annotation for the Posts table in a rails application using SQlite for the database.

```ruby
# <rails-lens:schema:begin>
# table = "posts"
# database_dialect = "SQLite"
# rest of annotation ommitted for brevity
# <rails-lens:schema:end>
```

The annotation also lets us know about the columns and indexes of the Posts table under the `columns` and `indexes` sections. Each of the columns of the Post table in the annotation is displayed with either `nullable = false` or `nullable = true` highlighting whether for the value of a given column can be `null` or not at the time of saving a post in the database. 

```ruby
# columns = [
#   { name = "id", type = "integer", primary_key = true, nullable = false },
#   { name = "title", type = "text", nullable = true },
#   { name = "body", type = "text", nullable = true },
#   { name = "category_id", type = "integer", nullable = false },
#   { name = "user_id", type = "integer", nullable = false },
#   { name = "created_at", type = "datetime", nullable = false },
#   { name = "updated_at", type = "datetime", nullable = false }
# ]
#
# indexes = [
#   { name = "index_posts_on_user_id", columns = ["user_id"] },
#   { name = "index_posts_on_category_id", columns = ["category_id"] }
# ]
```
The `foreign_keys` section of the annotation highlights the associations between the Post model and other models in the application namely the User and Category. 

We get extra notes as part of the annotation as well and this is where `rails-lens` shines compared to other annotation gems. 

As part of the output you can see that `rails-lens` notified us of some ways in which the schema and therefore our application could use some improvements. This includes the N+1 query risk associated with the Comment model.

```ruby
# - Association 'comments' has N+1 query risk. Consider using includes/preload
```

Also you might remember that aas part of the columns section of the annotation, the title and body columns had `nullable` set to true which means that its possible to create a post without a title or a body. This is not ideal as we would want every post within the application to have a title and a body. Rails lens lets us know about this as well. 

```ruby
# - Column 'title' should probably have NOT NULL constraint
# - Column 'body' should probably have NOT NULL constraint
```
And now that we know about it, we can proceed to address it whether by adding a validation or any other approach. 

Lastly Rails lens lets us know of the potential impacts on performance we many run into if we have so many records in the application and it doesn't stop at that, it tells us what could potentially cause performance problems and what we can do to solve this. 

```ruby
# - Large text column 'title' is frequently queried - consider separate storage
# - Large text column 'body' is frequently queried - consider separate storage
```
And there you have it, we were able to get information about the models in the application, all without constantly going back and forth between the model files and the `schema.rb` file.

The annotated Comment, User and Category models are attached below.

```ruby
# <rails-lens:schema:begin>
# table = "users"
# database_dialect = "SQLite"
#
# columns = [
#   { name = "id", type = "integer", primary_key = true, nullable = false },
#   { name = "name", type = "string", nullable = true },
#   { name = "email", type = "string", nullable = true },
#   { name = "created_at", type = "datetime", nullable = false },
#   { name = "updated_at", type = "datetime", nullable = false }
# ]
#
# == Notes
# - Column 'name' should probably have NOT NULL constraint
# - Column 'email' should probably have NOT NULL constraint
# - String column 'name' has no length limit - consider adding one
# - String column 'email' has no length limit - consider adding one
# - Column 'email' is commonly used in queries - consider adding an index
# <rails-lens:schema:end>


class User < ApplicationRecord
end
```

```ruby
# <rails-lens:schema:begin>
# table = "comments"
# database_dialect = "SQLite"
#
# columns = [
#   { name = "id", type = "integer", primary_key = true, nullable = false },
#   { name = "content", type = "text", nullable = true },
#   { name = "post_id", type = "integer", nullable = false },
#   { name = "created_at", type = "datetime", nullable = false },
#   { name = "updated_at", type = "datetime", nullable = false }
# ]
#
# indexes = [
#   { name = "index_comments_on_post_id", columns = ["post_id"] }
# ]
#
# foreign_keys = [
#   { column = "post_id", references_table = "posts", references_column = "id" }
# ]
#
# == Notes
# - Association 'post' should specify inverse_of
# - Column 'content' should probably have NOT NULL constraint
# - Large text column 'content' is frequently queried - consider separate storage
# <rails-lens:schema:end>


class Comment < ApplicationRecord
  belongs_to :post
end
```

```ruby
# <rails-lens:schema:begin>
# table = "categories"
# database_dialect = "SQLite"
#
# columns = [
#   { name = "id", type = "integer", primary_key = true, nullable = false },
#   { name = "name", type = "string", nullable = true },
#   { name = "created_at", type = "datetime", nullable = false },
#   { name = "updated_at", type = "datetime", nullable = false }
# ]
#
# == Notes
# - Column 'name' should probably have NOT NULL constraint
# - String column 'name' has no length limit - consider adding one
# <rails-lens:schema:end>


class Category < ApplicationRecord
end
```
Also, its possible to annotate specific model files using the command below;

```ruby
bundle exec rails_lens annotate --model Post Comment
```
To remove the annotations, run the command below:

```ruby
bundle exec rails_lens remove
```


