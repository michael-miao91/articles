还是直接进入主题，大致就是这样的一个场景。在购物车中我需要删除某一条商品信息，但是当我点击删除的时候。我并不想通过form表单的形式，去在后台写相应的函数。而且在查找相应资料的时候，发现`form`表单中的`method`属性只有`get`和`post`两种，那么对于`method=delete`这种情况，又该如何解决呢？所以，想到了用`javascript`，通过对相应的button添加javascript事件，那么我就可以直接在js中去调用后端的API。虽然这两种方法实现的效果(删除以及刷新页面)差不多，但是因为之前对js接触很少，还是感觉非常的奇妙。所以还是把这个问题的一些思路记录下来，记录自己在解决这个问题中学到了一点东西。

对于购物车中的删除某条数据的按钮，html如下

```html
<button name="removeFromCart" type="submit" class="btn btn-primary remove" data-item="${item.itemId}">Remove From Cart</button>
```
所以接下来就需要对这个按钮添加相应得的click事件，js代码如下所示

```javascript
<script type="text/javascript">
    $(".remove").click(function(){
        var itemId = $(this).data("item");
        var data = {
            email: 'user@freewheelers.com',
            itemId: itemId
        };
        $.ajax({
            method: 'POST',
            url: '/shoppingCart/remove',
            data: data,
            success: function(data) {
                if (data === true) {
                    location.reload(true);
                }
            }
        });
    });
</script>
```

在button的html代码中，有这样一个属性`data-item="${item.itemId}"`, 为什么这样做呢？因为如果在这个购物车中有多条商品信息，那么当点击删除按钮时，又怎么知道移除的是哪一条数据呢？所以这里button绑定了itemId这样一个字段，想要获的这个字段，只需要通过`var itemId = $(this).data("item");`这条语句就可以达到目的。注意，这里是通过class属性获取的button信息，因为当我尝试用id属性去获取button信息的时候，在有多条语句的情况下，我只能得到第一条记录的js事件。

第二点就是如何在js对于后端发起一个请求，看上面的代码直接是用到了`$.ajax()`这样一个方法，这个方法里面有许多的参数，具体的参数可以参看相应的[API文档](http://api.jquery.com/jQuery.ajax/).当我们在做到这一步的时候，就遇到了一个问题。遇到了各种各样的http错误，那么又该如何正确调用后台的方法呢？相应的java代码如下

```java
@RequestMapping(value = {"/remove"}, method = RequestMethod.POST)
@ResponseBody
public boolean removeFromShoppingCart(Model model,@RequestParam(value = "email") String email, @RequestParam(value = "itemId") int itemId ) {
    Account account = accountService.getAccountIdByEmail(email);
    Item itemToRemove = itemService.get(Long.valueOf(itemId));
    shoppingCartService.removeFromUserShoppingCart(account, itemToRemove);
    updateCartQuantity(model,account);
    return true;
}
```

其实这段代码还是有很多的瑕疵，我觉得下面的一段代码可能会显得更好一点，都是完成这样的一个功能(可能场景还是有一点点不同)，如下所示：

```java
@RequestMapping(value = "{id}", method = RequestMethod.DELETE, produces = "application/json")
    @ResponseBody
    public boolean delete(@PathVariable int id) {
        userService.deleteUser(id);
        return true;
    }
```

当事情做到这一步的时候，似乎觉得一切都大功告成了。但是当你用一些restful client工具测试的时候，或许你会遇到`http406`这样的错误。所以怎么去解决呢？其实也是很简答，因为在spring mvc中，你还需要去配置一些相应的依赖，让你的项目能够支持`json`这种数据的传递，而本项目是采用的`gradle`这个项目构建工具, 所以Google一下就知道答案了。在[这个网页中](http://stackoverflow.com/questions/25265407/add-boon-or-jackson-json-to-android-studio-with-gradle)提供了下面的解决思路。在`build.gradle`这个文件中，加入下面的依赖包即可。

```
compile(
     [group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: '2.4.1'],
     [group: 'com.fasterxml.jackson.core', name: 'jackson-annotations', version: '2.4.1'],
     [group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.4.1']
)
```

至此，就完成了这个问题的大概解决方案。
