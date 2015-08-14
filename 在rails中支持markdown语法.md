之前利用octopress平台搭建博客，写作的方式就是利用`markdown`语法写作。只不过在当时，应该是markdown和html两种格式的混合，还是显得相当的不专业。尽管最后，花了一些时间回头来在修改之前的文件。但是在链接，以及图片，甚至标题上，还是用的html语法。

在自己搭建这个博客平台的时候，开始的时候就一直在考虑。我应该要支持markdown语法，因为作为一个技术博客来说，支持markdown格式，第一显的更为的专业，第二它对于代码的显示更为的赏心悦目。所以，其实在没开始搭建博客的时候。就试着用一个例子来摸索一下，rails支持markdown语法是否会很复杂？

其实，还是按照以前的一种经验。如果要增加一个新的rails的功能，首先google一下是否有相应的gem包支持。果不其然，这里我们是用到了`redcarpet`这样一个gem。之后，我们需要做的就是通读一下其在[github](https://github.com/vmg/redcarpet)上的一个大体介绍。如果还是不太清楚的话，可以参考一下其他资料。我这里找到的是网上的一个教程[markdown格式化内容](http://happypeter.github.io/rails-tricks/10_markdown.html)

之后核心代码就是在`ApplicationHelper.rb`中添加下面的一段代码：

```ruby
def markdown(text)
  renderer = Redcarpet::Render::HTML.new(hard_wrap: true, filter_html: true)
  options = {
    autolink: true,
    no_intra_emphasis: true,
    fenced_code_blocks: true,
    lax_html_blocks: true,
    strikethrough: true,
    superscript: true
  }
  Redcarpet::Markdown.new(renderer, options).render(text).html_safe
end
```

之前对于这段代码我也只是单纯的拷贝而已，并不理解它为什么要这样做？后来再回过头来读了一下github上的文档。看到了下面的一段代码：

```ruby
markdown = Redcarpet::Markdown.new(renderer, extensions = {})
```

那么，我们所需要做的就是实例化renderer对象以及extensions这个数组对象，这里也就是出现了上面的一大段代码的缘由。

同时，为什么需要在`ApplicationHelper.rb`中声明这一段代码呢？在上一篇文章中我也是给过假设，因为调用方法我是在view页面中，而rails执行的一个逻辑就是，通过controller再到view页面，那么view页面需要调用方法，就只有在调用在helper中申明的方法。当然我这里又是做了两个实验：
1. 去掉在`ApplicationHelper.rb`中的方法的实现，转而将其写在`ArticleHelper.rb`中，程序还是能够正常运行
2. 去掉在`ArticleHelper.rb`中方法的实现，转而将其写在任何一个helper文件中，程序都是可以正常运行

以上的设置，就已经实现了markdown的功能，我们接下来做的就只是在输出的地方调用这个方法即可。
那么接下来的问题就来了，我怎样才能支持语法高亮呢？

这里还是需要用到一个gem，名字就做[pygments.rb](https://github.com/tmm1/pygments.rb)
在其文档上，有下面的一句代码

```ruby
Pygments.highlight(File.read(__FILE__), :lexer => 'ruby')
```

这样我们就实现了语法高亮，那么怎样才能在rails中实现呢？

还是参照了上面的一段代码，在`ApplicationHelper.rb`中声明如下（完整代码）：

```ruby
module ApplicationHelper
  class HTMLwithPygments < Redcarpet::Render::HTML
    def block_code(code, language)
      Pygments.highlight(code, lexer: language)
    end
  end

  def markdown(text)
    renderer = HTMLwithPygments.new(hard_wrap: true, filter_html: true)
    options = {
        autolink: true,
        no_intra_emphasis: true,
        fenced_code_blocks: true,
        lax_html_blocks: true,
        strikethrough: true,
        superscript: true
    }
    Redcarpet::Markdown.new(renderer, options).render(text).html_safe
  end
end
```

要注意，这里renderer的申明是用到了`pygments.rb`中的方法。那么接下来需要做的就是，设置一下相关的css。注意这里又是新建了一个css文件，里面有下面的一句代码：

```ruby
<%= Pygments.css(:style => "monokai") %>
```

为什么有上面的代码呢？可以参考一下[这个网址](http://richleland.github.io/pygments-css/),
它里面其实有各种各样的css样式，我们参考一种即可。这样就解决了在rails中如何支持markdown以及语法高亮的功能。