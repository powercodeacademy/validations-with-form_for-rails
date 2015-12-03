# Validations with `form_for`

Now that we know Rails automatically performs validations defined on models,
let's use this information to easily display validation errors to the user.

# Objectives

After this lesson, you'll be able to...

- use form_for to display a form with Validations
- print out full error messages above the form
- customize aspects of the text_field error message if possible

# `form_for`

This step will make heavy usage of `form_for`, the high-powered alternative to
`form_tag`. The biggest difference beteen these two helpers is that `form_for`
creates a form specifically **for** a model object. `form_for` is full of
convenient features.

A basic implementation looks like this:

```erb
<!-- app/views/posts/new.html.erb //-->

<%= form_for @post, url: {action: "create"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit "Create" %>
<% end %>
```

To wire up an empty form in our `new` view, we need to create a blank object:

```ruby
# app/controllers/posts_controller.rb

  def new
    @post = Post.new
  end
```

Here's our usual vanilla `create` action:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.create(post_params)

    redirect_to post_path(@post)
  end
```

We still have to solve the dual problem of what to do when there's no valid
model object to redirect to, and how to maintain validation feedback state while
still re-rendering the same form.

# Re-Rendering With Errors

Remember from a few lessons ago how CRUD methods return `false` when validation
fails? We can use that to our advantage here and branch our actions based on the
result:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.new(post_params)

    if @post.save
      redirect post_path(@post)
    else
      render :new
    end
  end
```

# Full Messages with Prepopulated Fields

Because of `form_for`, Rails will automatically prepopulate the `new` form with
the values the user entered on the previous page.

To get some extra verbosity, we can add the snippet from the previous lesson to
the top of the form:

```erb
<!-- app/views/posts/new.html.erb //-->

<% if @post.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@post.errors.count, "error") %>
      prohibited this post from being saved:
    </h2>

    <ul>
    <% @post.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

