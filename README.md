# Displaying Has Many Through In Rails

## Objectives

1. Construct a bi-directional has_many through.
2. Query for associations via the belongs_to, has_many, and has_many through associations.
3. Iterate over associations in a view and display associated data for a primary instance.
4. Identify the join model in a has_many through.

## Overview

You can use simple associations to display data to users in Rails, but more complex relationships are just as easy thanks to Active Record and `has_many, through`.

## Lesson

### has_many, through

Suppose you're making a blog and want users to sign up and comment on posts. A comment belongs to a post, and a post has many comments. A user has many comments, and a comment belongs to a user. This is straightforward.

The relationship between a user and the posts they've commented on is many-to-many. We set up a many-to-many relationship using a join table. In this case, `comments` acts as the join table. Any table with two foreign keys can be a join table. A row in the `comments` table might look like this:

| id | content           | post_id | user_id |
|----|-------------------|---------|---------|
| 1  | "I loved this post!" | 5       | 3       |

This shows that the `Comment` with ID `1` was created by the `User` with ID `3` for the `Post` with ID `5`. You can determine all posts a user has commented on and all users who commented on any post. You can call `@user.posts` to get all those posts.

To set this up, you'll need migrations for `comments`, `posts`, and `users` tables. Migrations and models are included in this repo.

```ruby
# db/migrate/xxx_create_posts.rb
class CreatePosts < ActiveRecord::Migration[7.1]
  def change
    create_table :posts do |t|
      t.string :title
      t.string :content
      t.timestamps null: false
    end
  end
end
```

```ruby
# db/migrate/xxx_create_users.rb
class CreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      t.string :username
      t.string :email
      t.timestamps null: false
    end
  end
end
```

```ruby
# db/migrate/xxx_create_comments.rb
class CreateComments < ActiveRecord::Migration[7.1]
  def change
    create_table :comments do |t|
      t.string :content
      t.belongs_to :user
      t.belongs_to :post
      t.timestamps null: false
    end
  end
end
```

### Models

```ruby
class User < ApplicationRecord
  has_many :comments
  has_many :posts, through: :comments
end

class Post < ApplicationRecord
  has_many :comments
  has_many :users, through: :comments
end

class Comment < ApplicationRecord
  belongs_to :post
  belongs_to :user
end
```

Notice that we can't just declare that our `User` `has_many :posts` because our `posts` table doesn't have a foreign key called `user_id`. Instead, we tell Active Record to look through the `comments` table to figure out this association by declaring that our `User` `has_many :posts, through: :comments`. Now, instances of our `User` model respond to a method called `posts`. This will return a collection of posts that share a comment with the user.

### Displaying Comments on Our Posts

Now that our association is set up, let's display some data. First, let's set up our `Post#show` page to display all of the comments on a particular post. We'll include the username of the user who created the comment as well as a link to their show page.

In `app/controllers/posts_controller.rb`, define a `show` action that finds a particular post to make it available for display.

```ruby
# app/controllers/posts_controller.rb

class PostsController < ApplicationController

  def show
    @post = Post.find(params[:id])
  end
end
```

In our `Post#show` page, we'll display the title and content information for the post as well as the information for each comment associated with the post.

```erb
# app/views/posts/show.html.erb

<h2><%= @post.title %></h2>
<p>
  Content: <%= @post.content %>
</p>
Comments:
  <% @post.comments.each do |comment| %>
    <%= link_to comment.user.username, user_path(comment.user) %> said
    <%= comment.content %>
  <% end %>
```

This is the same as we've done before –– we're simply looking at data associated with posts and comments. Calling `comment.user` returns for us the `User` object associated with that comment. We can then call any method that our user responds to, such as `username`.

### Adding Posts to Our Users

Let's say that on our `User#show` page we want our users to see a list of all of the posts that they've commented on. What would that look like?

Because we've set up a join model, the interface will look almost identical. We can simply call the `posts` method on our user and iterate through.

```erb
# app/views/users/show.html.erb

<h2><%= @user.username %> </h2> has commented on the following posts:

<% @user.posts.each do |post| %>
  <%= link_to post.title, post_path(post) %>
<% end %>
```

## Conclusion

Displaying data via a `has_many, through` relationship looks identical to displaying data through a normal relationship. That's the beauty of abstraction –– all of the details about how our models are associated with each other get abstracted away, and we can focus simply on the presentation.
