问题：

在使用mdl的过程中，发现其有一些js特效是没有做react兼容的，官网的文档上说当react升级到2.0的时候会实现兼容，在现在版本下，只能自己去实现兼容性代码。

索性在github有人已经实现了mdl的react组件化，在一个 [react-mdl](https://github.com/tleunen/react-mdl) 的项目中，demo看起来很好用，但是在实际使用过程中，原来的gulp构建代码无法实现构建。

因为react-mdl是使用es6+的react编写方式，使用的是es6的模块依赖实现，而我现在的项目是使用的是browerify进行的模块化，构建代码是不兼容的。

这个问题gulp中也有babel可以解决，它可以先将最新es6的模块依赖方式编译成browerfify支持的模块依赖方式，使用的是babelify。当然babel本身很强大，babelify的实现也是使用了babel-core的node api。使用的时候针对react [ES7+ Property Initializers](https://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html)的语法，需要配置babelify的属性。

具体的gulp模块化的任务如下：

    gulp.task(tasks.moduleJs,[tasks.cleanJs], function() {
        return browserify({
            entries: [config.jsPath + 'main.js'], // Only need initial file, browserify finds the deps
            transform: [babelify.configure({ optional: [
                "es7.classProperties"
            ]}),reactify] // We want to convert JSX to normal javascript
        })
        .bundle() // Create the initial bundle when starting the task
        .pipe(source('main.js'))
        .pipe(gulp.dest(config.dest));
    });

顺便说一下：这里的reactify实现了react中jsx语法的预编译工作，我们就不需要在浏览器端进行jsx的解析了。
