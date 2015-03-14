对于这个博客站点，之前一直就是在本地开发、运行。当时也是以为到时候直接在heroku上部署就可以了，但是在自己真正去实施的时候，还是遇到了许多坑，甚至让自己一度想要放弃。

###OpenShift
首先因为在本地开发的项目数据库是用到的mysql，但是在heroku上如果要用到mysql，就必须绑定信用卡。所以，因此就转到了红帽的[OpenShift](https://www.openshift.com/)这个平台来部署.但是，因为自己没有做过太多深入的了解。所以，很显然的部署失败了。然后自己就放弃了这样的一种思路，不得不说，这也算是自己的一种失败。如果首先坚定了这种方案，就应该好好地实施下去。不能因为一点点的失败和困难，就想着放弃。

###heroku
之后很显然，又回过头来打heroku的注意。因为heroku中，可以用`postgresql`这样一个免费的数据库。所以，这里的解决方案就是首先开始对项目所用的一个数据库框架更改。

1.首先在`Gemfile`文件中，

```Bash
- gem 'mysql2'
+ gem 'pg' 
```

2.执行`bundle install`命令，然后更改`database.yml`文件。本项目的配置如下：

```Bash
default: &default
  adapter: postgresql
  encoding: utf8
  pool: 5

development:
  <<: *default
  database: blog_development

test:
  <<: *default
  database: blog_test

production:
  <<: *default
  database: blog_production
```

原则上更改了这些配置信息之后，就可以在heroku上部署了，但是在本机上到这里就遇到了各种各样的挑战。

####postgresql连接不上
因为我在本地启动`rails server`后，服务器会报一个错误。

```Bash
Can't connect to the postgres server ls: /tmp/.s.PGSQL.5432: No such file or directory
```

google上查了各种各样的解决方案，但是一直无法解决。对于此，真的是一度崩溃。甚至我还以为是heroku客户端的问题，因为它报了一个warning，说本机的git版本过低。那么如何更新git呢？下了git最新安装包，都无济于事。而且用`brew upgrade git`,会显示git正在使用的版本，无法更新。最后解决git的更新，还是用到了如下的命令。

```Bash
brew update
brew upgrade git
```

然后解决之前的postgresql无法连接的问题。我试着用`initdb /usr/local/var/postgres`这条命令，后台报了一个目录已存在的错误。到这里真的是没有其他办法了，一咬牙直接删除了这个目录，再重新执行此命令，启动`rails s`,会惊奇的发现，这个问题就阴差阳错的解决了。

那么最后的一个步骤，就是在heroku上部署了。我之前的做法就是，把代码直接上传到github中去。然后在heroku中直接连接github，接着deploy。想法是很好地，同时我也看到了deploy的log显示已经成功。但是当我访问网址的时候，就一直报一个500错误。一直尝试着解决，却一直找不到正确的途径。虽然，想到了是没有执行数据库的迁移`rake db:migrate`。但是，却不知道在哪里去运行这个命令。

也是google各种相关的解决方案，却总是找不到一种好的解决办法。最后找到了下面的一种方法，因为在`heroku git`这块区域内，它给出了下面的代码

```Bash
$ heroku git:clone -a projectName
```
但是当我去clone这个代码库的时候，却一直是一个空的项目。那么，这里就得出了一个结论。*我们的代码其实没有提交到`heroku git`上*。似乎也是离成功越来越近了，那么怎么样才能把代码提交到heroku上呢？我这里用上了下面的操作，就把代码直接push到了heroku上了。

```Bash
heroku create tsglxt
git push heroku master
```

所以，接下来的事情也是很简单了。把heroku中的代码直接clone下来，然后就可以执行各种操作了。比如说
`heroku run rake db:migrate`,它会直接对应到云端上去。

###snap-ci
如果仅仅只是前面的那些配置，那么就需要很多的人为操作。所以，我这里也是想办法把snap-ci加上去了。通过snap-ci和heroku的集成，我这里就可以达到自动部署的目的了。同时也是直接把
`github --> snap-ci --> heroku`这条线成功的串联起来。我可以直接在本地写代码，完成项目的开发，当我把代码上传到github后，snap-ci会自动完成相关的部署，并把代码push到heroku上。然后到时候，我直接访问云端的网页即可。

