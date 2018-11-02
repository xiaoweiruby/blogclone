# 使用 Ruby on Rails 构建一个博客系统

```
10468  cd newspace
10469  ls
10470  rails new blogclone
10471  cd blogclone
10472  git init
10473  git add .
10474  git commit -m "initial commit"
10483  git remote add origin https://github.com/xiaoweiruby/blogclone.git
10484  git push -u origin master
```
# 一/页面结构美化
```
Gemfile

gem 'better_errors', '~> 2.4'
gem 'bulma-rails', '~> 0.6.1'
gem 'simple_form', '~> 3.5'

app/assets/stylesheets/application.scss
 @import "bulma";
```

```
10476  git checkout -b add_gems
10477  atom .
10478  bundle install
10479  rails generate simple_form:install
10480  git add .
10481  git commit -m "add gems"
10482  git push origin add_gems
```
# 二/添加文章数据
```
10488  git checkout -b add_posts
10489  rails g controller posts
10490  rails g model Post title:string content:text
10492  rake db:migrate
10493  rails s
10494  git add .
10495  git commit -m "add_posts & layouts"
10496  git push origin add_posts
```
app/views/layouts/application.html.erb
```
<!DOCTYPE html>
<html>
  <head>
    <title>DemoBlog</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
  	<section class="hero is-primary is-medium">
	  <!-- Hero head: will stick at the top -->
	  <div class="hero-head">
	    <nav class="navbar">
	      <div class="container">
	        <div class="navbar-brand">
	          <%= link_to 'Demo Blog', root_path, class: "navbar-item" %>
	          <span class="navbar-burger burger" data-target="navbarMenuHeroA">
	            <span></span>
	            <span></span>
	            <span></span>
	          </span>
	        </div>
	        <div id="navbarMenuHeroA" class="navbar-menu">
	          <div class="navbar-end">
	            <%= link_to "Create New Post", new_post_path, class:"navbar-item" %>
	          </div>
	        </div>
	      </div>
	    </nav>
	  </div>

	  <!-- Hero content: will be in the middle -->
	  <div class="hero-body">
	    <div class="container has-text-centered">
	      <h1 class="title">
	        <%= yield :page_title %>
	      </h1>
	    </div>
	  </div>
	</section>
    <%= yield %>
  </body>
</html>

```
app/controllers/posts_controller.rb
```
class PostsController < ApplicationController

    def index
        @posts = Post.all.order("created_at DESC")
    end

    def new
        @post = Post.new
    end

    def create
        @post = Post.new(post_params)

        if @post.save
            redirect_to @post
        else
            render 'new'
        end
    end

    def show
        @post = Post.find(params[:id])
    end

    def update
        @post = Post.find(params[:id])

        if @post.update(post_params)
            redirect_to @post
        else
            render 'edit'
        end
    end

    def edit
        @post = Post.find(params[:id])
    end

    def destroy
        @post = Post.find(params[:id])
        @post.destroy

        redirect_to posts_path

    end

    private

    def post_params
        params.require(:post).permit(:title, :content)
    end

end

```
```
config/routes.rb

Rails.application.routes.draw do
  resources :posts
  root 'posts#index'
end

```
app/views/posts/-form.html.erb
```
<div class="section">
<%= simple_form_for @post do |f| %>
  <div class="field">
    <div class="control">
      <%= f.input :title, input_html: { class: 'input' }, wrapper: false, label_html: { class: 'label' } %>
    </div>
  </div>

  <div class="field">
    <div class="control">
      <%= f.input :content, input_html: { class: 'textarea' }, wrapper: false, label_html: { class: 'label' }  %>
    </div>
  </div>
  <%= f.button :submit, 'Create new post', class: "button is-primary" %>
<% end %>
</div>
```
app/views/posts/index.html.erb
```
<% content_for :page_title,  "Index" %>

<div class="section">
	<div class="container">
		<% @posts.each do |post| %>
			<div class="card">
		  <div class="card-content">
		    <div class="media">
		      <div class="media-content">
		        <p class="title is-4"><%= link_to post.title, post  %></p>
		      </div>
		    </div>
		    <div class="content">
		     	<%= post.content %>
		    </div>
        <div class="comment-count">
		    	<span class="tag is-rounded"><%= post.comments.count %> comments</span>
		    </div>
		  </div>
		</div>
		<% end %>
	</div>
</div>

```
app/views/posts/show.html.erb
```
<% content_for :page_title, @post.title %>

<section class="section">
	<div class="container">
		<nav class="level">
		  <!-- Left side -->
		  <div class="level-left">
		    <p class="level-item">
		        <strong>Actions</strong>
		    </p>
		  </div>
		  <!-- Right side -->
		  <div class="level-right">
		  	<p class="level-item">
		    	<%= link_to "Edit", edit_post_path(@post), class:"button" %>
		  	</p>
		  	<p class="level-item">
				<%= link_to "Delete", post_path(@post), method: :delete, data: { confirm: "Are you sure?" }, class:"button is-danger" %>
				</p>
		  </div>
		</nav>
		<hr/>

		<div class="content">
			<%= @post.content %>
		</div>
	</div>
</section>
```
app/views/posts/new.html.erb
```
<% content_for :page_title, "Create a new post" %>
<%= render 'form' %>
```
app/views/posts/edit.html.erb
```
<% content_for :page_title, "Edit Post" %>
<%= render 'form' %>

```
# 三/添加评论数据
```
10497  git checkout -b add_comments
10498  rails g controller comments
10499  rails g model Comment name:string comment:text
10500  rails db:migrate
10501  rails g migration add_post_id_to_comments post_id:integer
10503  rake db:migrate
10504  rails s
10505  git add .
10506  git commit -m "add_comments"
10507  git push origin add_comments
```
app/controllers/comments_controller.rb
```
class CommentsController < ApplicationController

    def create
        @post = Post.find(params[:post_id])
         @comment = @post.comments.create(params[:comment].permit(:name, :comment))
        redirect_to post_path(@post)
    end

    def destroy
        @post = Post.find(params[:post_id])
        @comment = @post.comments.find(params[:id])
        @comment.destroy
        redirect_to post_path(@post)
    end

end

```
# 四、关系的对应关系
```
app/models/post.rb
class Post < ApplicationRecord
  has_many :comments, dependent: :destroy
end

app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :post
end

```
# 五、评论模板页面
app/views/comments/-form.html.erb
```
<%= simple_form_for([@post, @post.comments.build]) do |f| %>
<!--
collection.build(attributes = {}, …) Returns one or more new objects of the collection type that have been instantiated with attributes and linked to this object through a foreign key, but have not yet been saved. Note: This only works if an associated object already exists, not if it‘s nil!
-->
<div class="field">
  <div class="control">
    <%= f.input :name, input_html: { class: 'input' }, wrapper: false, label_html: { class: 'label' } %>
  </div>
</div>

<div class="field">
  <div class="control">
    <%= f.input :comment, input_html: { class: 'textarea' }, wrapper: false, label_html: { class: 'label' }  %>
  </div>
</div>
<%= f.button :submit, 'Leave a reply', class: "button is-primary" %>
<% end %>

```
app/views/comments/-comment.html.erb
```
<div class="box">
  <article class="media">
    <div class="media-content">
      <div class="content">
        <p>
          <strong><%= comment.name %>:</strong>
          <%= comment.comment %>
        </p>
      </div>
    </div>
     <%= link_to 'Delete', [comment.post, comment],
                  method: :delete, class: "button is-danger", data: { confirm: 'Are you sure?' } %>
  </article>
</div>
```
