其实说到注册和登录的问题，其实之前也是写过一篇博客简单的描述过这样的一个过程。但是因为博客的迁移，但是我并没有把之前的博客迁移过来，而且之前所介绍的还是太过的简单。这里再好好地梳理一下rails的注册和登录到底是怎么样实现的？

###注册
要实现注册的功能，这里是用到了`bcrypt`这样一个gem包。之后在用户模型中，加入`has_secure_password`这样一个属性。在[这个网页中](http://railstutorial-china.org/book/chapter6.html)给出了下面的解释：
>1.在数据库中的 password_digest列存储安全的密码哈希值<br>
>2.获得一对“虚拟属性”，password和 password_confirmation，而且创建用户对象时会执行存在性验证和匹配验证<br>
>3.获得authenticate方法，如果密码正确，返回对应的用户对象，否则返回false

通过上面的方法，就已经大体上可以达到了一个注册的效果。因为所谓的注册，还是可以看做是一个新增某个对象的过程。但是这里还需要注意的是一个后台验证的环节。比如：密码不能为空，两次密码输入一致等。
如下所示：

```ruby
validates :password, confirmation: true
validates :password_confirmation, presence: true
```

为什么上面还判断了重复密码是否为空？我们可以看一下官方文档给的解释：
>只有 email_confirmation 的值不是 nil 时才会做这个验证。所以要为确认属性加上存在性验证

通过上面的操作和配置后，其实就可以实现一个用户注册的功能了。

###登录
首先我这里是利用`rails g controller sessions`新建了一个controller，然后就可以利用`new create`两个方法来实现登录的操作。

代码如下：

```ruby
class SessionsController < ApplicationController
  include SessionsHelper
  def new
    unless session[:user_id].nil?
      session.delete(:user_id)
    end
    @user = User.new
  end

  def create
    user = User.find_by(username: params[:username])
    if user && user.authenticate(params[:password])
      sign_in(user)
      redirect_to articles_path
    else
      render :new
    end
  end

  def destroy
    log_out
    redirect_to root_path
  end
end
```

其实这里登录的操作并不复杂，因为前面也是介绍过，这个用户模型它是自带了一个authenticate方法。利用这个方法就很好地实现了，判断某个用户是否是合法用户。这里其实主要想解释的前面我写了一个`include SessionsHelper`这样的一个语句，因为我这里调用了`sign_in(user)`和`log_out`这两个方法，但是这两个方法的实现我是写在了`SessionsHelper.rb`中。所以，如果我调用它里面的方法就必须在前面进行一个引用的申明。

当然，我这里也是做了下面两个实验。
1.我如果把这几个方法的申明写在`ApplicationHelper`，但是假如我不对其申明，同样的服务器会报下面的一个错误。

```html
undefined method `sign_in' for #<SessionsController:0x007fe153510860>
```

2.但是这里如果我把这几个方法的申明写在`ApplicationController`,即使我不对其做相关的申明，程序还是能够正确的执行。

3.这里还有一个很有意思的地方，在`Application.html.erb`中我调用了一个方法块

```ruby
<% if current_user %>
<% end %>
```

但是这个方法的申明，我却是写在了`ApplicationHelper`中，如果我把这个方法写在`ApplicationController`中，服务器就报了下面的一个错误。

```ruby
undefined local variable or method `current_user' for #<#<Class:0x007fe153358a68>:0x007fe153e08a58>
```

这样，是否可以大胆的假设一下。如果是在某个controller中调用其它文件的方法，就应该引入相应的文件。
如果是在某个view界面，就可以直接调用其helper文件中的方法。（如果调用controller中的方法，甚至会出错）

