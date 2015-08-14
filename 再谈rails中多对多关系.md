为什么要这样说呢？其实，之前早些时候。也是写了一篇博文来说明，来阐述rails中的多对多关系。但是，好像在那时，因为考虑的不太周全。所以，其实有着很多的漏洞。这次，结合之前做的一个例子，做一个相对完善的总结。

首先，说一下它的一个大致的情景。文章和标签两者之间的关系，他们是属于多对多的关系，即`一篇文章可能有多个标签，一个标签也可能属于多个文章`。基于此，我们已经创建好了两个model，`article`和`tag`。那么接下来的操作，就是怎样将两者之间的一个多对多的关系建立起来。

我们都知道，如果建立多对多的关系，就应该依靠第三张表来关联。在rails中，[官网文档](http://guides.ruby-china.org/association_basics.html)中给出了如下的说明：

>has_many :through 关联经常用来建立两个模型之间的多对多关联。这种关联表示一个模型的实例可以借由第三个模型，拥有零个和多个另一个模型的实例。

所以，在这里我们可以这样建立它们三个的关系。

```ruby
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.string :title
      t.text :content
      t.timestamps
    end
  end
end

class CreateTags < ActiveRecord::Migration
  def change
    create_table :tags do |t|
      t.string :name
      t.timestamps
    end
  end
end

class CreateArticleTags < ActiveRecord::Migration
  def change
    create_table :article_tags do |t|
      t.belongs_to :article
      t.belongs_to :tag
      t.timestamps
    end
  end
end
```

之后，我们在schema文件中，可以看到相关的建表语句：

```ruby
create_table "article_tags", force: true do |t|
    t.integer  "article_id"
    t.integer  "tag_id"
    t.datetime "created_at"
    t.datetime "updated_at"
  end
```
这说明数据库表中已经是有了相应的字段，那么我们接下来就应该做的是。怎样在模型中将它们关联起来？
按照官方文档的说明，相应的代码如下：

```ruby
class Article < ActiveRecord::Base
    has_many :article_tags, dependent: :destroy
    has_many :tags, through: :article_tags
end

class Tag < ActiveRecord::Base
    has_many :article_tags, dependent: :destroy
    has_many :articles, through: :article_tags
end

class ArticleTag < ActiveRecord::Base
  belongs_to :article
  belongs_to :tag
end
```

我原来以为其实进行了这样的配置之后，而且我在后台敲入如下的代码进行操作：

```ruby
t = Tag.first
t.articles
```

会显示一个空数组或者相应的文章数据，所以，我觉得其实就已经配置成功了。但是当我尝试，对`article_tags`这张表添加新的数据，它的外键引用并不存在。但是，结果显示，它还是能够插入进去的。
所以，这里就应该加上相应的验证。

在`model/article_tag`中加上下面的两句代码：

```ruby
validates :article, presence: true
validates :tag, presence: true
```

PS：其实在rails的这种关联关系中，它并没有自己想象的那么复杂。甚至它还可以总结为以下很简单的几个步骤。
1. 在数据库中建立相应的字段
2. 在model中进行关联
3. 在model中添加相应的验证（级联删除，存在性验证）