AngularJS&mdash;&mdash;起步
===


##引入 Angular &lt;script&gt; 标签
    <!doctype html>
    <html xmlns:ng="http://angularjs.org" ng-app>
        <body>
            ....
            <script src="angular.js">
        </body>
    </html>
1. 将&lt;script&gt;标签放置到页面底部。将&lt;script&gt;放在页面底部是为了改善页面加载时间，因为在加载`angular.js`的时候不会堵塞HTML DOM的加载。你可以在[http://code.angularjs.org ](http://code.angularjs.org)获得最新的版本。
> *    `angular-[version].js` 是一个可读的大文件，它适合开发和调试使用。
> *    `angular-[version].min.js`是一个压缩、混淆过的文件，适合产品发布。    

2. 将`ng-app`放置到应用程序的根。如果你期望angular自动启动你的程序，通常是放置到`<html>`标签上。

3. 如果使用IE7，需添加`id="ng-app"`。

4. 如果期望在IE上使用旧的语法指令`ng:`，则需要在`<html>`标签上包含一个xml命名空间。(出于历史原因，不再推荐使用`ng:`)    


##自动启动

Angular会在`DOMContentLoaded`事件或`angular.js`脚本加载完成并且`document.readyState`被设置为`complete`时自动初始化。Angular在初始化时会查找`ng-app`指令指定的应用根。如果`ng-appa`指令被找到，则执行如下操作：
> * 为指令加载相关模块。
> * 创建应用的[injector](https://docs.angularjs.org/api/auto/service/$injector)。
> * 编译、处理指令`ng-app`指定的DOM。也就是说将这个DOM元素作为应用的一部分来处理。    

    <!doctype html>
    <html ng-app="optionalModuleName">
	<body>
	    I can add: {{ 1+2 }}.
	    <script src="angular.js"></script>
	</body>
    </html>

##手动初始化

如果在初始化过程中你需要更多的控制，你可以使用一个手动启动方法来替换。例如，当你需要一个脚本加载器或着在angular编译页面之前执行一个操作。    
下面是一个手动启动的例子：    

    <!doctype html>
    <html>
    <body>
      Hello {{'World'}}!
      <script src="http://code.angularjs.org/angular.js"></script>

      <script>
        angular.module('myApp', [])
          .controller('MyController', ['$scope', function ($scope) {
            $scope.greetMe = 'World';
          }]);

        angular.element(document).ready(function() {
          angular.bootstrap(document, ['myApp']);
        });
      </script>
    </body>
    </html>