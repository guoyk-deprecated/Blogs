---
layout: post
title: '[Rails] Transit Between Rails Projects Smoothly Using Symbolic Link'
---

由于年少无知的时候，写的Rails项目使用了太多诡异的Hack，并且没有慎重考虑就上了`AngularJS`，导致项目维护难度日益增大。

之后我新开了一个坑，为了让两者能够平滑过渡，我使用了`git submodule`和 `软链` 将二者结合起来。

首先，我将旧的项目作为一个`git submodule`放在新的项目里面，然后用软链直接将旧项目的一部分链接到新项目里面。

git处理软链的时候只是将其作为单一普通文件处理，不会迭代进入软链目录内；而Docker在处理软链的时候是完全忽略的，需要预先处理好坑。

当前我将以下几个目录通过软链共享了：

```
db/
app/models/
app/uploaders/
app/jobs/
```

数据库的Schema和Migration必须要共享。然后，我通过重型的Model层，将Controller轻量化，从而使这种方式的连接更加高效。

When I was young and innocent (LOL.), my Rails project depends on too much hacks, and I move to `AngularJS` in lack of consideration. These problems lead to a huge unmaintainable project.

So I move on to next generation of project. To make a smooth transition, I use `git submodule` and `symbol link` to combine them together.

First, I add old project into new project as a submodule.

Git will regard symbol link file as a plain file, it will not iterate into it.

Docker will completely ignore symbol link.

Currently I'v combine these folders together:

```
db/
app/models/
app/uploaders/
app/jobs/
```

Share of `schema.rb` and `db/migrations` is a must.
I structured a heavy Model layer, and lightened Controller layer, which make sharing `app/models` more valuable.
