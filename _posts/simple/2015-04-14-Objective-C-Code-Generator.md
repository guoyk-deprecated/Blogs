---
layout: post
title: "[iOS] 自己写Objective C代码生成工具"
---

最近在写 iOS 的时候遇到了一些重复代码，但是由于 Objective C 的语言特性没法进行完整封装的情况，于是，我想到了写一个代码生成器。

代码生成器的概念其实和服务器渲染HTML的概念一样，最终生成的代码由两部分决定

* 模板
* 数据

明白了这一点就可以很快地写出符合需要的代码生成器。

以我当前的需求为例，我使用了 JSONModel 进行 JSON 映射，因此需要写大量的 Model 定义，内容的核心就是枯燥地把字段和类型定义出来，维护和查看都很累。

于是我便决定写出一个通用的工具来实现类似的需求。

### 第一步，选定模板引擎

我必然选了`erb`

为什么？

第一，我喜欢 Ruby，Mac 平台内置 Ruby 环境

第二，`erb` 是通用模板，像`slim`,`jade`之类的专门为`HTML`设计的模板语言自然不行

### 第二部，写脚本

我是这样设想的，脚本从一个固定的文件夹里读取数据，然后遍历整个目录，把所有名叫 `AAA.h.erb` 的文件看做模板，与数据结合渲染出 `AAA.h` 文件。

首先我准备了一个目录叫`Nova`，在项目根目录下，里面有两个文件夹，一个叫 `Data`，里面放了一堆需要当做数据用的 `YAML` 文件，另外一个叫 `Helpers`,里面的所有 `Ruby` 文件都会被 `require` 方便在模板中调用。

上代码

```ruby
#!/usr/bin/env ruby

require 'yaml'
require 'erb'
require 'digest'
require 'ostruct'

# Method to render a template with data
def render_erb(template, vars)
  ERB.new(template).result(OpenStruct.new(vars).instance_eval { binding })
end

# Extract source root
src_root = File.expand_path('../../', __FILE__)
nova_root= File.expand_path('../../Nova', __FILE__)

# Load All Helpers
Dir["#{nova_root}/Helpers/*.rb"].each do |file|
  puts "Helper Loaded: #{file}"
  require file
end

# Find all *.erb and render from yml file
Dir["#{src_root}/**/*.erb"].each do |file|
  erb     = File.open(file).read
  idf     = File.basename(file).split('.').first
  target  = File.join File.dirname(file), File.basename(file, ".erb")
  data    = YAML.load_file "#{src_root}/Nova/Data/#{idf}.yml"
  sha1    = Digest::SHA1.hexdigest(data.to_s)

  result  = render_erb(erb, { data: data, sha1: sha1 })
  File.open(target, 'w') { |tf| tf.write(result) }
  puts "File Rendered: #{target}"
end

puts "Done !"
```

### 第三步，让脚本自动运行

这个很简单，把脚本放在合适的位置，注意相对位置关系，在 Xcode Build Phrase 里面新建一个 `Run Script`，然后把脚本位置放进去就好了。

### 分享一些我用到的模板

### JSONModel 生成器

`Nova/Data/Models.yml` 里面放上需要生成的 Class 的定义,格式如下

```yaml
ClassA:
  id: number, required, index
  property_b: ClassB
ClassB:
  id: number, required, index
  property_c: string, ignored
```

就可以了。

Models.h.erb

```erb
#import <JSONModel/JSONModel.h>

//  前置声明

<% data.keys.each do |clazz| %>
@class <%= clazz %>;
<% end %>

<% data.keys.each do |clazz| %>
@protocol <%= clazz %> @end
<% end %>

//  定义
<% data.keys.each do |clazz| %>
@interface <%= clazz %> : JSONModel
<% data[clazz].each do |key, config| %>
<%= gen_jsonmodel_property(key, config) %>
<% end %>
@end
<% end %>
```

Models.m.erb

```erb
#import "Models.h"

<% data.keys.each do |clazz| %>
@implementation <%= clazz %> @end
<% end %>
```

JSONModelPropertyGenerator.rb (放在 `Helpers/`)

```ruby
def gen_jsonmodel_property(key, value)
  configs = value.split(',').map do |str|
  str.strip
  end
  type = configs.shift
  type_mod = []

# Type Transform
  if type == 'string'
  type = 'NSString'
  end
  if type == 'number'
  type = 'NSNumber'
  end
  if type == 'number'
  type = 'NSNumber'
  end
  if type == 'array'
  type = 'NSArray'
  end
  if type.start_with? 'array<'
  type_mod << type.match(/array<(.+)>/)[1]
  type = 'NSArray'
  end

# Check modifier
  configs.each do |mod|
  raise "Not accepted modifier: #{mode}" unless ['required','index', 'ignored'].include? mod
  end

# Modifier
  if configs.include? 'ignored'
  type_mod << 'Ignored'
  end
  if configs.include? 'index'
  type_mod << 'Index'
  end
  unless configs.include? 'required'
  type_mod << 'Optional'
  end

# Join type with mod
  if type_mod.length > 0
  type += "<#{type_mod.join(',')}>"
  end

  "@property (nonatomic, strong) #{type} * #{key};"
  end
```

#### 字符串混淆

`Nova/Data/SecureStringProvider.yml` 里面放上`key-value`格式的需要进行混淆的字符串,最终生成的`SecureStringProvider`就会有一堆公用方法可以调用，获取到这些`String`。

现在我做的这个还缺乏一些对于Slash的处理，当然一般情况下够用。

SecureStringProvider.h.erb

```erb
#import <Foundation/Foundation.h>

#define SSP SecureStringProvider

@interface SecureStringProvider : NSObject

<% data.keys.each do |key| %>
+ (NSString*)<%= key %>;
<% end %>

@end
```

SecureStringProvider.m.erb

```erb
#import "SecureStringProvider.h"

@implementation SecureStringProvider

<% data.keys.each do |key| %>
+ (NSString*)<%= key %> {
  static NSMutableString * _cache;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
      _cache = [NSMutableString stringWithCapacity: 100];<% data[key].each_char do |car| %>[_cache appendString: @"<%= car %>"];<% end %>
      });
  return _cache;
}
<% end %>

@end
```
