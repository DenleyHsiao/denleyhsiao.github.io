---
layout: post
title:  "rails启动分析(1) --- rails 命令"
date:   2015-11-21 8:25:00
categories: rails gem
---

学习应该讲究方法，这样可以事半功倍，本文通过一个老手的角度，在转入新的领域时，是如何快速学习的（注：后文中的老手代指熟悉C++的人，而新手代指
准备学习Ruby语言的人，其实学习方法都是想通的，跟语言无关)。

##选择很重要
####选择怎么开始
万事开头难，在没有老师或项目的环境下，只能是看书，俗话说的好：“读书破万卷, 下笔如有神”，但时间有限，“万卷”中选择看哪些书又成了问题，
怎么办?

* 选择经典的：找baidu, 逛论坛 
* 选择英文版的：原因我就不说了，前提是你的英语还行（英语不行也不能入这行啊）

我目前选择的是：

* 《Programming Ruby中文版第二版》：经典，而且它是中文版（我的英语不行，没办法，入错行了），但只支持1.8.7的RUBY
* 《Pragmatic.rogramming.Ruby.1.9.and.2.0.4th.Edition》，上本书的第四版，原因是它支持较新的RUBY（1.9/2.0），但是英文版

####为什么选择分析rails
* 首先，它是一个GEM，一个在目前很火的GEM，这样分析起来就会很有成就感；
* 然后，GEM相当于一个隔离层，让调用方与被调用方的关注分离，只要保证向外提供的API一致就可以了：实现方只要关注如何优化实现，而调用方只要关注
自己的业务；
* 最后，就是Ruby中的GEM自动化程度很好，轻松几步就能生成自己的demo, 然后发布到rubygems网站上，他人就可以立即使用你的成果。

可能你要问了，选择这么重要，我又怎么知道目前选择学习的方向是对的呢，呵呵，前面不是说过“读书破...”, 这里说下我选择的理由吧

* 第一，因为我是老手，以前学习时栽过跟头，又对DLL有过深入学习，有没有发现，在封装方面，GEM是不是有些像DLL（在跨语言调用方面，目前还知道）；
* 然后，我也是在没法选择的情况下，先学RUBY语法，也是学得枯燥，在看到GEM这章时就有动手的冲动---这是好的学习境界，但前面的经历也要必须的

##方法更重要
####多动手
学习语言最重要的是学以致用，对于新手，强烈建议边看书边对示例进行验证，但对于我这样一个从C++转入Ruby的老手，这一招实在是很难坚持下去，怎么办呢？

办法还是有的：先利用老手的优势，把Ruby的语法过一遍，呵呵，一看都还懂，只是有点枯燥（没办法，前面不是说了入错行了吗）---如果没耐心全看的话，
可以与你熟悉的比较，着看其区别就可以了。这个RUBY网站就做的不错，从其它语言转过来的老手可以往这看：[老手必看](ruby-from-other-langauges)。
但基本上没恒心去一个一个在RUBY环境中验证，怎么办呢？

####多动脑

##开始分析
####运行分析
* 首先准备好环境，安装ruby, rails:
<pre><code>rvm install ruby-2.2.2
rvm use ruby-2.2.2 --default
gem install rails
</code></pre>

* 运行下，看效果：
<pre><code>rails new demo</code></pre>

* rails为什么可以运行起来：
<pre><code>which rails </code></pre>
输出可执行文件rails的绝对路径,这个文件所在的路径又是怎么找到的呢?

* 检查下全局路径：
<pre><code>echo $PATH </code></pre>
输出的里面是不是有rails所在的路径

* rails这个文件是怎么来的呢：
<pre><code>gem install rails</code></pre>
GEM 在我们进行安装时自动生成的（最终运行的是gem包中的bin目录下同名可执行文件）

* 其所在的目录又是怎么加到$PATH中呢：
<pre><code>rvm use ruby-2.2.2 --default</code></pre>
安装RUBY时， 可以通过以下切换后的变化来验证：
<pre><code>rvm install ruby-2.2.0(如果没安装）
rvm use ruby-2.2.0
echo $PATH
</code></pre>

####源码分析
* 现在开始看看rails文件内容
{% highlight ruby linenos %}
require 'rubygems'

version = ">= 0.a"

# 使用命令行指定要求运行的railties版本（如：rails _4.2.4_ s）
if ARGV.first
  str = ARGV.first
  str = str.dup.force_encoding("BINARY") if str.respond_to? :force_encoding
  if str =~ /\A_(.*)_\z/ and Gem::Version.correct?($1) then
    version = $1
    ARGV.shift
  end
end

# 根据版本号运行程序
gem 'railties', version
load Gem.bin_path('railties', 'rails', version)
{% endhighlight %}

* 跟踪第16行的gem方法，进入keneral_gem.rb
首先说明下，这里的gem方法同与Gemfile中的gem方法相同，不过后者完成下载、部署、以及载入
{% highlight ruby linenos %}
  def gem(gem_name, *requirements) # :doc:
    ...
    # *requirements参数带入版本要求的正则表达式
    dep = Gem::Dependency.new(gem_name, *requirements)

    loaded = Gem.loaded_specs[gem_name]

    return false if loaded && dep.matches_spec?(loaded)

    # 确定gem_name使用的版本
    spec = dep.to_spec

    Gem::LOADED_SPECS_MUTEX.synchronize {
      # 下文剖析
      spec.activate
    } if spec
  end
{% endhighlight %}
进入gem方法中的dep.to_spec,到了dependency.rb文件中：
{% highlight ruby linenos %}
  def matching_specs platform_only = false
    matches = Gem::Specification.stubs.find_all { |spec|
      self.name === spec.name and # TODO: == instead of ===
        requirement.satisfied_by? spec.version
    }.map(&:to_spec)

    if platform_only
      matches.reject! { |spec|
        not Gem::Platform.match spec.platform
      }
    end
    
    # 通过排序，最新的版本就是last了
    matches.sort_by { |s| s.sort_obj } # HACK: shouldn't be needed
  end
  
  def to_specs
    matches = matching_specs true

    # TODO: check Gem.activated_spec[self.name] in case matches falls outside
    # 错误处理
    if matches.empty? then
      ...
      raise error
    end

    # TODO: any other resolver validations should go here

    matches
  end

  def to_spec
    matches = self.to_specs

    active = matches.find { |spec| spec.activated? }

    return active if active

    matches.delete_if { |spec| spec.version.prerelease? } unless prerelease?
    # 哈哈，找到了，使用最新版本
    matches.last
  end
{% endhighlight %}
原来，dep.to_spec方法通过正则匹配出来的版本号，最终确定了{gem_name}所在的目录

* 进入gem方法中的spec.activate
{% highlight ruby linenos %}
     ##
     # Activate this spec, registering it as a loaded spec and adding
     # it's lib paths to $LOAD_PATH. Returns true if the spec was
     # activated, false if it was previously activated. Freaks out if
     # there are conflicts upon activation.
   
     def activate
       other = Gem.loaded_specs[self.name]
       if other then
         check_version_conflict other
         return false
       end
   
       raise_if_conflicts
   
       activate_dependencies
       add_self_to_load_path
   
       Gem.loaded_specs[self.name] = self
       @activated = true
       @loaded = true
   
       return true
     end
{% endhighlight %}
前面{gem_name}目录确定，这里就是将此目录下的lib目录就可以加入$LOAD_PATH, 最终点亮后续所有有依赖此GEM的“require '{gem_name}'"

* 分析完gem方法，现在就乘load方法了
由于前一gem方法，此处就容易懂了："load/require '{gem_name}'"就可以定位到相应的{{gem_name}}.rb文件载入了，这里是railties.rb文件了---这才是
真正的启动程序
     
* 下面说下版本号与文件名分离的好处：

> 1. 能适应版本号统一在Gemfile中，隔离变化，并能像全面一样动态生成全路径
> 2. 其它依赖中引用时不用写"require 'rails-4.2.5’", 同样隔离变化

[ruby-from-other-langauges]: https://www.ruby-lang.org/en/documentation/ruby-from-other-languages/