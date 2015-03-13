---
layout: post
title: '[Rails] 以正确的姿势重定向 Remote Form'
---

一般情况下，创建一个资源之后会302重定向到这个新创建的资源，比如创建一个帖子：

```
POST /posts   -> GET /post/111
```

或者

```
POST /posts   -> GET /posts
```

在Rails中，普通表单会重新加载整个页面，失去了 Turbolinks ，或者说 pjax 的优雅。

如果使用 remote 表单的话，所有处理逻辑必须使用 `ajax:success` 等一系列 jQuery 事件来处理。

对于上述第二种重定向，如果重定向的目的地址和源地址是一样的话，使用统一的封装会方便很多。

我是这样写的:

```coffee
 $(document).on 'page:change', ()->
   # Bind form with page reloads
   $("form.need-reload").on 'ajax:success', (data, status, xhr)->
     Turbolinks.visit()
   $("a.need-reload").on 'ajax:success', (data, status, xhr)->
     Turbolinks.visit()
```

简单来说就是给这个remote表单加上一个类，在js中为所有这个类的表单增加ajax钩子，请求成功之后就刷新页面。

Controller里面就可以渲染为空了:

```ruby
render nothing: true
```

当我想要以类似的方法实现第一种重定向的时候，我写成了这个样子:

```ruby
redirect_to action: :show, id: @post.id
```

对应的js里面加上了对于302的处理, 用`Turbolinks.visit(url)`访问新的页面。

**但是，我错了，大错特错**

jQuery 是会自动追踪302的，也就是说渲染出302后，jQuery会追踪302获取重定向之后的页面。

js里面永远不会捕获到302。

正确的方法是，如果请求要求的mime是js，直接渲染一段js调用`Turbolinks.visit(url)`指向新的页面；如果是html, 使用`redirect_to`。

代码如下:

```ruby
 respond_to do |format|
         format.js do
           if @post.parent_post.present?
             render js: "Turbolinks.visit('#{url_for(action: :show, id: @post.parent_id, jump_sub_post: @post.id )}')"
           else
             render js: "Turbolinks.visit('#{url_for(action: :show, id: @post.id)}')"
           end
         end
         format.html do
           if @post.parent_post.present?
             redirect_to action: :show, id: @post.parent_id, jump_sub_post: @post.id
           else
             redirect_to action: :show, id: @post.id
           end
         end
       end
```

或许有更好的解决办法。
