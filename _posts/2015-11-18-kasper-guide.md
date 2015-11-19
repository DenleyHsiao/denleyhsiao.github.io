---
layout: post
title:  "kasper使用问题"
date:   2015-11-18 10:15:00
categories: miscellaneous
---

一直在找合适的主题，也觉得[掌心中的demo][zhangxin]主题效果不错，但最终决定使用kasper，还是因为看了Rei的[简单就好][simple-is-better],
不过初次使用kasper，就遇到了下面的问题：

#### 安装完成后，启动jekyll时出现如下错误
<pre><code>
 Incremental build: disabled. Enable with --incremental
      Generating...
       Deprecation: Collection#map should be called on the #docs array directly.
                    Called by /Users/xiao_dli/work/kasper/_plugins/rssgenerator.rb:46:in `block in generate'.
       Deprecation: Collection#count should be called on the #docs array directly.
                    Called by /Users/xiao_dli/work/kasper/_plugins/rssgenerator.rb:49:in `rescue in block in generate'.
       Deprecation: Collection#reverse should be called on the #docs array directly.
                    Called by /Users/xiao_dli/work/kasper/_plugins/rssgenerator.rb:51:in `block in generate'.
       Deprecation: Document#title is now a key in the #data hash.
                    Called by /Users/xiao_dli/work/kasper/_plugins/rssgenerator.rb:53:in `block (3 levels) in generate'.
       Deprecation: Document#excerpt is now a key in the #data hash.
                    Called by /Users/xiao_dli/work/kasper/_plugins/rssgenerator.rb:55:in `block (3 levels) in generate'.
</code></pre>
最后发现原来是不能使用jekyll新的版本3.0.0（3.0.1也不行），通过以下方式解决
<pre><code>
gem uninstall jekyll
gem install jekyll -v '<3.0.0'
</code></pre>

######附：
* [采用Jekyll + github + pygments构建个人博客的最终说明](http://www.jianshu.com/p/609e1197754c)
* [Jekyll指南](http://jekyll.bootcss.com/docs/home/)
* [Jekyll变量 和 Jekyll模板语法教程](http://higrid.net/c-art-jeklly_template_data.htm)
* [Jekyll 语法简单笔记](http://ju.outofmemory.cn/entry/98471)
* [Markdown 语法说明(中文版)](http://wowubuntu.com/markdown/)

[zhangxin]: http://www.zhanxin.info/themes.html
[simple-is-better]: http://chloerei.com/2015/10/21/simple-is-better/
