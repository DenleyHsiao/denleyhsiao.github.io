---
layout: post
title:  "Ruby与C++交互指南"
date:   2015-11-21 10:25:00
categories: ruby cpp rice swig
---

C++与Ruby交互主要有两种方式：
* 通过Rice
* 通过SWIG

##通过Rice进行交互
####安装
<pre><code>gem install rice
</code></pre>

####使用

GOOGLE了一下，关于使用的看的有点乱，无法完整做出一个demo, 还是先从[源码](https://github.com/jasonroelofs/rice)中的示例(sample目录)入手吧，
操作步骤如下
<pre><code>ruby extconf.rb
make
ruby test.rb
</code></pre>

####自己动手
这里主要是测试调用C++的代码

* 生成extconf.rb文件

```ruby
require 'mkmf-rice'
#其中sample是sample.cpp的文件名，可任意定义(最好与要公开的RUBY类同名,如这里是test,暂时命名是以示区别)
create_makefile('sample')
```

* 生成sample.cpp文件

```cpp
#include "rice/Class.hpp"
#include <cassert>
#include <iostream>

using namespace Rice;

void hello(const char* name)
{
  assert(name != NULL && strlen(name) != 0);
  std::cout<<"Hello, "<< name << "!" <<std::endl;
}

//函数名最后字符格式：必须是".当前文件名"
extern "C" void Init_sample()
{
  //Test为最终RUBY看到的类名
  Class tmp_ = define_class("Test")
    .define_method("hello", &hello);
}
```

* 生成test.rb
```ruby
require_relative 'sample'
t = Test.new
t.hello("World!")

* 按全面的步骤运行就OK了

## 通过SWIG方式交互


####附：
* [在 C++ 中使用 Rice 构建 Ruby 扩展](http://www.ibm.com/developerworks/cn/opensource/os-extendruby/index.html#list1)
