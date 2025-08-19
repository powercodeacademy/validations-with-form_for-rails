# Validations with form_for

> **Note:** In the latest versions of Rails in Nitro at the time of writing this, the recommended way to build forms is with [`form_with`](https://guides.rubyonrails.org/form_helpers.html#deprecating-form-for-and-form-tag), which replaces both `form_for` and `form_tag`. However, many legacy codebases (including this lesson) still use `form_for`. For forms that work with models, `form_for` is much better than `form_tag`, but for new code, you should use `form_with`.

Now that we know Rails automatically performs validations defined on models, let's use this information to easily display validation errors to the user.

## Objectives

After this lesson, you'll be able to...

- use `form_for` to display a form with Validations
- print out full error messages above the form


## The differences between `form_for` and `form_tag`

This lesson uses `form_for`, which is much more powerful and convenient than `form_tag` when working with model objects. `form_for` automatically handles many details for you, such as routing, field naming, and prepopulating values. However, in new Rails projects, you should use `form_with`, which is the new standard and combines the best features of both helpers.

In the example below, `@post` is the model object that needs a form. `form_for` automatically performs a route lookup to find the right URL for post.


`form_for` takes a block. It passes an instance of [FormBuilder](http://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html) as a parameter to the block, which is what `f` is below. Again, for new code, prefer `form_with`, but understanding `form_for` is important for working in existing Rails codebases.

A basic implementation looks like this:

```erb
<!-- app/views/posts/edit.html.erb //-->

<%= form_for @post do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

This creates the HTML:

```html
<form
  class="edit_post"
  id="edit_post"
  action="/posts/1"
  accept-charset="UTF-8"
  method="post"
>
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input type="hidden" name="_method" value="patch" />
  <input
    type="hidden"
    name="authenticity_token"
    value="nRPP2OqVKB00/Cr+8EvHfYrb5sAkZRtr8f6dzBaJAI+cMceR0fUatcLWd4zdwYCpojW2J3QLK6uyBKeFAgZvmw=="
  />
  <input
    type="text"
    name="post[title]"
    id="post_title"
    value="Existing Post Title"
  />
  <textarea name="post[content]" id="post_content">
Existing Post Content</textarea
  >
  <input type="submit" name="commit" value="Update Post" />
</form>
```

Here's what we would need to do with `form_tag` to generate the exact same HTML:

```erb
<!-- app/views/posts/edit.html.erb //-->

<%= form_tag post_path(@post), method: "patch", name: "edit_post", id: "edit_post" do %>
  <%= text_field_tag "post[title]", @post.title %>
  <%= text_area "post[content]", @post.content %>
  <%= submit_tag "Update Post" %>
<% end %>
```

`form_tag` doesn't know what action we're going to use it for, because it has no model object to check. `form_for` knows that an empty, unsaved model object needs a `new` form and a populated object needs an `edit` form. This means we get to skip all of these steps:

1. Setting the `name` and `id` of the `<form>` element.
2. Setting the method to `patch` on edits.
3. Setting the text of the `<submit>` element.
4. Specifying the root parameter name (`post[whatever]`) for every field.
5. Choosing the attribute (`@post.whatever`) to fill for every field.


Nifty! Just remember: for new Rails code, use `form_with` instead of `form_for` or `form_tag`.

## Using `form_for` to generate empty forms

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

We still have to solve the dual problem of what to do when there's no valid model object to redirect to, and how to hold on to our error messages while re-rendering the same form.

## Re-Rendering With Errors

Remember from a few lessons ago how CRUD methods return `false` when validation fails? We can use that to our advantage here and branch our actions based on the result:

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

## Full Messages with Prepopulated Fields

Because of `form_for`, Rails will automatically prepopulate the `new` form with the values the user entered on the previous page.

To get some extra verbosity, we can add the snippet from the previous lesson to the top of the form:

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

## More Freebies: `field_with_errors`

Let's look at another nice feature of `FormBuilder`. Here's our `form_for` code again:

```erb
<!-- app/views/posts/edit.html.erb //-->

<%= form_for @post do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

The `text_field` call generates this tag:

```html
<input
  type="text"
  name="post[title]"
  id="post_title"
  value="Existing Post Title"
/>
```

Not only will `FormBuilder` pre-fill an existing `Post` object's data, it will also wrap the tag in a `div` with an error class if the field has failed validation(s):

```html
<div class="field_with_errors">
  <input
    type="text"
    name="post[title]"
    id="post_title"
    value="Existing Post Title"
  />
</div>
```

This can also result in some unexpected styling changes because `<div>` is a block tag (which takes up the entire width of its container) while `<input>` is an inline tag. If your layout suddenly gets messed up when a field has errors, this is probably why.


## Recap

`form_for` gives us a lot of power when working with model-backed forms in legacy Rails codebases!

While `form_for` is still important to understand for maintaining and working in existing projects, remember that `form_with` is the new standard for building forms in modern Rails applications. For new code, you should use `form_with`, but knowing how `form_for` works will help you navigate and update older codebases.

Our challenge as developers is to keep track of the different layers of magic that make these tools so convenient. The old adage is true: we're responsible for understanding not only _how_ to use `form_for` but also _why_ it works. Otherwise, we'll be completely lost as soon as a sufficiently unusual edge case appears.

When in doubt, **read the HTML**. Get used to hitting the "View Source" and "Open Inspector" hotkeys in your browser (`Ctrl-u` and `Ctrl-Shift-i` on Chrome Windows; `Option-Command-u` and `Option-Command-i` on Chrome Mac), and remember that most browsers let you [examine `POST` data in their developer network tools](http://superuser.com/questions/395919/where-is-the-post-tab-in-chrome-developer-tools-network).
