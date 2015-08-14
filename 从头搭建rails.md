虽然在之前搭建博客的时候，也是走了一遍是如何用rails来搭建一个程序的相关流程。虽然是大同小异，但是当自己又重新捋一遍的时候，还是多多少少的碰到了一些坑。还是趁着头脑稍微有一点印象，将其好好地整理一下，以便在下次搭建的时候，有一个参照和依据。

首先，我们做的第一步就是，初始化一个rails项目。因为这里我使用bower来下载相关的css和js文件，并且用grunt来做进一步的处理。相应的步骤就是，通过`bower init`和`bower install`来安装相应的依赖包。

那么这里第一个坑来了，当我在项目中增加`Gruntfile.js`和`package.json`。但是，当我在终端执行`grunt`的时候，后台总是会报一个错误。

```html
Fatal error: Unable to find Gruntfile.
```

开始我以为是没有执行`npm install`导致的，但是当我执行了这一句语句后，一直还是这样的一个样子。反正有点不知道是咋回事吧，之后又重新新建一个项目。按照如下的步骤一步一步的执行，竟然成功了。

```html
touch package.json Gruntfile.js
npm install
grunt
```

不过，这里新建两个文件并不是直接通过用鼠标右键那种方式新建，而是通过相应的命令。

相应的package.json和Gruntfile.js文件可以参考如下：
####package.json

```html
{
  "name": "boostrapDemo",
  "description": "Example project to demonstrate Grunt.",
  "version": "0.1.0",
  "private": true,
  "devDependencies": {
    "grunt": "^0.4.5",
    "grunt-cmd-concat": "~0.2.0",
    "grunt-cmd-transport": "~0.2.0",
    "grunt-concat-css": "^0.3.1",
    "grunt-contrib-clean": "~0.4.0",
    "grunt-contrib-uglify": "~0.2.0"
  }
}

```

####Gruntfile.js

```html
module.exports = function(grunt){
    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        uglify: {
            options: {
                banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
            },
            static_mappings:{
                files:[
                    {
                        src: './bower_components/bootstrap/dist/js/bootstrap.js',
                        dest: './app/assets/javascripts/bootstrap.min.js'
                    }
                ]
            }
        },
        concat_css: {
            all: {
                src: ["./bower_components/bootstrap/dist/css/bootstrap.css"],
                dest: "./app/assets/stylesheets/bootstrap.min.css"
            }
        }
    });
    // 加载提供"uglify"任务的插件
    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-concat-css');

    // 默认任务
    grunt.registerTask('default', ['uglify', 'concat_css']);
}
```

回过头来想了一下，或许是哪里的顺序颠倒了才会导致这样的一个错误。

之后，就是rails在heroku的部署了。这里其实可以参考上一篇的博文，我觉得当我们在做到这一步的时候。应该想到的是，我应该怎样在远端执行`rake db:migrate`语句。这样才能建立起相应的数据库连接，基于这个思路，去执行，然后就可以了。