---
title: ruby中的三大框架——rails,sinatra,grape
date: 2015-11-28T00:00:00.000Z
author: kirineko
draft: false
summary: ruby中的三大框架——rails,sinatra,grape
tags:
  - ruby
createTime: 2015-11-28T00:00:00.000Z
permalink: /article/xppbtrqu/
---

>> 10年前，Java业界有着三大框架————Struts，Spring，Hibernate.时光飞逝，当年风靡一时的技术慢慢淡出了人们的视野，Spring MVC，Mybatis等技术开始大行其道，然而，这都不是重点，作为一个非Java技术圈的程序喵，今天要讨论的主题缺失Ruby中的三大框架。

# Ruby也有三大框架？

Ruby确实不像Java那样，声名显赫，三大框架互相配合，天衣无缝。Because We Only Need One Fullstack Framework————Rails! 嗯，全栈框架Rails的推出，颠覆性地改变了Web开发界。本次我们就来Discuss一下Ruby中的几个框架。

# Ruby Basic
首先还是介绍一下Ruby，简单的说Ruby具有如下几个特点:

1. 动态语言

	所谓动态语言是指语言数据类型可以动态改变，类型非常灵活，代表是PHP，Ruby，Python，JavaScript等脚本语言；与之相对的，Java、C++、C#等都是静态语言。动态语言的特点就是简单、灵活、多变、容易开发。
2. 完全面向对象

	Ruby的对象系统借鉴了Smalltalk的对象系统，具有完全面向对象的特点，在ruby中，类也是对象，对象本身也是对象。
3.  函数化

	Ruby的另一个原型是Lisp，甚至有人把Ruby比作是Lisp的一种现代方言。

4. 元编程和DSL

	正是由于以上几个特点，导致Ruby本身具有了强大的元编程和DSL能力。使用Ruby可以开发出非常奇特的程序和框架。

# Ruby Code
下面来欣赏一段Ruby Code！

```ruby
class Grade

  def initialize
    @dataStore = []
  end

  def add(score)
    @dataStore.push(score)
  end

  def avg
    sum = 0
    @dataStore.each do |data|
      sum = sum + data
    end
    avg = sum / @dataStore.length
    return avg
  end
end
```
该代码的功能是计算学生的平均分，使用面向对象技术构建，首先我们定义了一个类Grade，然后定义了构造函数initialize，在构造函数中，定义了一个实例变量`@dataStore`,该变量是一个数组。随后定义了两个方法add和avg，分别用来添加分数、求平均分，在添加分数函数中，调用数组@dataStore的push方法将score加入数组，在求平均分函数中，利用迭代器取出dataStore数组中的每一个元素，然后将其加入sum当中，最后用sum/length求出平均数。

>>以下部分现在没时间，先不写了，以后有时间再补充，代码先放着。

# Rails框架介绍


- 提到Ruby，人们首先会想到Ruby on Rails
- Rails是什么？
- Rails产生于2004年，是一个基于MVC的全栈Web开发框架，框架全部用ruby编写。

## 特点

- 三大要点：
  1. Ruby —— 语言（language）
  2. Gem —— 工具 （tools and library）
  3. Rails —— 平台 （platform）

- 两个基本原则
  1. DRY —— Don't Repeat Yourself
  2. CoC —— Convention over Configuration

- 一分钟搭建好Rails网站框架

``` bash
rails new "example_site"
bundle install
rails server
```
# Sinatra框架介绍

Rails本来是轻量级框架，但是随着ruby和rails的发展，框架开始变的越来越复杂。在这种状态下，一个更轻量级的框架产生了，它就是Sinatra.Sinatra是一个超轻量级Web框架，源代码总共只有1000多行，具有强大的可配置性。Sinatra ＝> Express.js

``` bash
gem install sinatra
```
``` ruby
require 'sinatra'
require 'sinatra/reloader'
require 'active_record'

ActiveRecord::Base.establish_connection(
  "adapter" => "sqlite3",
  "database" => "./bbs.db"
)

class Comment < ActiveRecord::Base
end

get '/' do
  @comments = Comment.order("id desc").all
  erb :index
end

post '/new' do
  Comment.create({:body => params[:body]})
  redirect '/'
end

post '/delete' do
  Comment.find(params[:id]).destroy
  redirect '/'
end
```
``` html
<h1>BBS</h1>
<ul>
  <% @comments.each do |comment| %>
  <li data-id="<%= comment.id %>">
    <%= comment.body %>
    <span class="deleteCmd" style="cursor:pointer;color:blue">[x]</span>
  </li>
  <% end %>
</ul>
<h2>Add New</h2>
<form method="post" action="/new">
  <input type="text" name="body">
  <input type="submit" value="post!">
</form>
```
# Grape框架介绍

- 产生目的：REST API > Web Site
- 一个微型框架，设计运行在Rack或配合Rails／Sinatra使用。
- API支持一些常见约束，常用于向客户端提供数据。
- 代表作品：Gitlab

- [example1](http://blog.elbowroomstudios.com/apis/)
- [example2](http://funonrails.com/2014/03/building-restful-api-using-grape-in-rails/)

``` ruby
class API < Grape::API
  prefix 'api'
  version 'v0.1', using: :path

  rescue_from ActiveRecord::RecordNotFound do |e|
    Rack::Response.new({
      error_code: 404,
      error_message: e.message
      }.to_json, 404).finish
  end

  rescue_from :all do |e|
    Rack::Response.new({
      error_code: 500,
      error_message: e.message
      }.to_json, 500).finish
  end

  mount Music::Store
  add_swagger_documentation api_version:'v0.1', mount_path: "/docs", markdown:GrapeSwagger::Markdown::KramdownAdapter

end
```

``` ruby
class API < Grape::API
  prefix 'api'
  version 'v0.1', using: :path

  rescue_from ActiveRecord::RecordNotFound do |e|
    Rack::Response.new({
      error_code: 404,
      error_message: e.message
      }.to_json, 404).finish
  end

  rescue_from :all do |e|
    Rack::Response.new({
      error_code: 500,
      error_message: e.message
      }.to_json, 500).finish
  end

  mount Music::Store
  add_swagger_documentation api_version:'v0.1', mount_path: "/docs", markdown:GrapeSwagger::Markdown::KramdownAdapter
end
```
