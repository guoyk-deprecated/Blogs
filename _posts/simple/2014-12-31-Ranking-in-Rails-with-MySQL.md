---
layout: post
title: '[Rails] Implement Ranking with MySQL'
---

契机是这样的，我这边负责的一个项目要做一个活动，有排行榜这样一个功能。

目前项目用的是阿里云的MySQL，MySQL没有很方便地做排行(Ranking)这样的功能，因此想了个极其简单的办法来实现。

首先，给模型上增加一个计数字段`happy15_count`，业务逻辑会触发更新这个字段。

然后，给模型写上一个方法，获取当前第几位。

```ruby

def happy15_count
  self.class.where("happy15_count > ? OR (happy15_count = ? AND id <= ?)", self.happy15_count, self.happy15_count, self.id).count
end

```

然后，获取排行榜的时候，就可以通过这样的方式实现了。

```ruby

Model.all.order("happy15_count DESC, id ASC").limit(100)

```

Our project is planning a promotion, which include a ranking.

So I come about a extreme easy way to implement that.

First, add a counter column to model, named, say `happy15_count`. Business logic will update this column.

Then, add a method which returns the rank of current model, like this

```ruby

def happy15_count
  self.class.where("happy15_count > ? OR (happy15_count = ? AND id <= ?)", self.happy15_count, self.happy15_count, self.id).count
end

```

Finally, the ranking could be written like this

```ruby

Model.all.order("happy15_count DESC, id ASC").limit(100)

```

Now, we have a ranking, and every model knows its position.
