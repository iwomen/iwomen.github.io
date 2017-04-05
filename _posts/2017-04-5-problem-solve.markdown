---
layout: post
title: Invalid GBK character "\xE2" 解决过程中的发现
date: 2016-02-15 15:32:24.000000000 +09:00
---
执行 $ bundle exec jekyll serve，之后出现这样的问题：
```
  Conversion error: Jekyll::Converters::Scss encountered an error
  while converting 'css/main.scss':
                    Invalid GBK character "\xE2" on line 10
jekyll 3.4.0 | Error:  Invalid GBK character "\xE2" on line 10
```

网上的解决方法：找到ruby的安装目录，里面也有sass模块，如这个路径：
```
C:\Ruby21-x64\lib\ruby\gems\2.1.0\gems\sass-3.4.8\lib\sass.rb
```
在这个文件里面engine.rb，添加一行代码
```
Encoding.default_external = Encoding.find('utf-8')
```
放在所有的require XXXX 之后即可。

之后的反馈是这样：

```
$ bundle exec jekyll serve
C:/Ruby23/lib/ruby/gems/2.3.0/gems/sass-3.4.23/lib/sass.rb:99:in `require': 
C:/Ruby23/lib/ruby/gems/2.3.0/gems/sass-3.4.23/lib/sass/engine.rb:1: 
syntax error,
unexpected tCONSTANT, expecting end-of-input (SyntaxError)
require 'set'Encoding.default_external = Encoding.find('utf-8')
```
之后继续做其他的事，然后不甘心。又仔细搜了些问答。找到了下面的内容：

在该路径文件里面 engine.rb，添加一行代码（放在所有的require XXXX 之后即可）：

```
require ...
require'sass/supports'
Encoding.default_external = Encoding.find('utf-8')
```

我忽然发现了天机。“放在所有的require XXXX之后”是个有歧义的句子。

一开始我的理解是

在每一个require XXXX 后面加一段```Encoding.default_external = Encoding.find('utf-8')```。

结果出现了上面提到的新问题。
```
unexpected tCONSTANT, expecting end-of-input (SyntaxError)
```

后来想到所有的require XXXX 之后也许是指在

```
{requireXXXX

requireXXXX

……

……

Require‘sass/supports’}

```
的后面只添加一段```Encoding.default_external = Encoding.find('utf-8')```。

也就是说在文档中找到Require‘sass/supports’然后在后面添加一段

```Encoding.default_external = Encoding.find('utf-8')```即可。

想通了这点，再试。运行成功。


