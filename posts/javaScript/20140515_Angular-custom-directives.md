AngularJS&mdash;&mdash;创建自定义指令(Directives)
===
<div style="font-size: 12px; color: #888; width:100%;text-align:left;margin-bottom:10px;">
作者  [爱看书不识字](http://blog.loafer.gitpress.org/)， 发布于2014年5月
</div>  

##什么是指令
从一个较高的层次上看，指令被标记在DOM元素上（如属性、元素名成、注释、或CSS 类），用来告诉Angular的HTML compiler 为DOM元素或者它的子元素附加特定行为。    

Angular内建了一组指令集，如`ngBind`、`ngModel`、`ngView`。就像创建controllers 和 services一样，你也可以创建自己的指令给Angular使用。当Angular启动应用时，HTML compiler会便利DOM树来匹配指令。

>什么是编译HTML模板？对于AngularJS来说，“编译”就是给HTML附加事件监听以便能作出响应。我们使用“编译”这个术语的原因，是因为这个附加指令的过程很像可编译语言编译源码的过程。    

<!--more-->
##匹配指令
在编写 directive 之前，我们需要知道HTML compiler是如何确定使用directive的。    
   
下面这个例子，我们可以说为`<input>`元素绑定了一个`ngModel`指令。 
  
    <input ng-model="foo">

下面也是绑定了一个`ngModel`指令： 
   
    <input data-ng:model="foo">

Angular使用元素标签和属性名来决定哪个元素匹配哪个 directive。我们通常使用区分大小写的 camelCase（驼峰）命名规则（如：`ngModel`）来引用指令。 然而，由于HTML是不区分大小写的，因此我们在DOM元素上使用小写形式，通常在DOM元素上使用以`-`为分割的属性（如:`ng-model`）。    

以下都是有效的：    
  1. 带有`x-`和`data-`前缀的元素或属性。    
  2. 使用`:`、`_`或`-`分割 来转换 camelCase（驼峰命名规则）。

这有一些与`ngBind`等价的相匹配的元素：    
script.js

    angular.module('docsBindExample', [])
      .controller('Controller', ['$scope', function($scope) {
        $scope.name = 'Max Karl Ernst Ludwig Planck (April 23, 1858 – October 4, 1947)';
      }]);

index.html

    <div ng-controller="Controller">
        Hello <input ng-model='name'> <hr/>
        <span ng-bind="name"></span> <br/>
        <span ng:bind="name"></span> <br/>
        <span ng_bind="name"></span> <br/>
        <span data-ng-bind="name"></span> <br/>
        <span x-ng-bind="name"></span> <br/>
    </div>

>最佳实践：推荐使用'-'分割（例如：`ng-bind`代替`ngBind`）。如果你想使用HTML验证工具，你可以使用`data-`前缀的版本来替换（如：`data-ng-bind`替换`ng-bind`）。前面提到的其他形式也可以，但是应避免使用。    

`$compiler`可以基于元素名称、属性、class或者注释来匹配指令。    

所有AngularJS提供的指令都是用过属性名、标签名、注释或class来匹配指令的。以下展示了模板引用指令的各种方式：

    <my-dir></my-dir>
    <span my-dir="exp"></span>
    <!-- directive: my-dir exp -->
    <span class="my-dir: exp;"></span>

>最佳实践：推荐使用标签名和属性名来使用指令，这要强于使用注释和class。这样做可以更简单的为元素匹配指令。       

>最佳实践：注释指令通常用于限制DOM API跨越多个元素创建指令（如`<table>`）。Angular1.2中介绍`ng-repeat-start`和`ng-repeat-end`可以很好的解决这个问题。开发人员应该尽可能的使用这个来代替自定义注释指令。    


##文本和属性绑定
在编译处理过程中，如果包含嵌入式表达式（embedded expressions）则使用[$interpolate](https://docs.angularjs.org/api/ng/service/$interpolate)匹配文本和属性。这些表达式被注册为[watches](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$watch)并作为[digest](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest)的一部分被更新。看下面的列子：
 
    <a ng-href="img/{{username}}.jpg">Hello {{username}}!</a>


##`ngAttr`属性绑定

有些属性浏览器是有限制的。    
例如，下面这个列子：

    <svg>
        <circle cx="{{cx}}"></circle>
    </svg>

我们原以为可以绑定成功，但是实际上我们会在控制台看到`Error: Invalid value for attribute cx="{{cx}}`输出。这是因为SVG DOM API不允许`cx="{{cx}}"`这种写法。    

使用`ng-attr-cx`可以解决这个问题。    

如果一个属性使用`ngAttr`前缀（或`ng-attr-`），在实际绑定过程中会被应用的相应的无前缀的属性上。
例如，你可以将上面的列子改为：

    <svg>
       <circle ng-attr-cx="{{cx}}"></circle>
    </svg>


##创建指令（Directives）

首先，我们讨论下 Directives 注册 API。就像 controllers，directive是注册在模型上的。你可以使用`module.directive`API注册一个指令。`module.directive`需要一个规范化的指令名成，后面跟一个工厂方法。这个工厂方法返回一个带有默认参数的对象，通过这个返回值对象告诉`$compiler`这个`directive`有什么样的行为。    

这个工厂方法仅在当`compiler`第一次匹配改指令的时候被调用。你可以在指令第一次被调用的时候做任何初始化工作。这个方法被[$injector.invoke](https://docs.angularjs.org/api/auto/service/$injector#invoke)调用，就像controller一样。    

>最佳实践：推荐工厂方法以返回Object的形式，替换返回函数。    

>最佳实践：为了避免与未来的标准冲突，最好在自定义指令名称前添加前缀。如果你创建了一个`<carousel>`指令，如果 HTML7中也存在一个相同的元素，那就会产生问题。前缀最好使用两个或三个字符。同样，不要使用`ng`作为前缀，这会影响Angular未来的指令。    


##Template-expanding directive
比如说，一个模板（译者注：就是指相同的html片段）代表一个用户信息。这个模板在你的代码中多个地方被使用。你想在某个地方修改它后，其他地方也跟着改变。要达到这个目的，最好的办法是给这个简单的模板创建一条指令（directive）。    

让我们使用静态模板来创建一条指令来替换它。    
script.js

    .controller('Controller', ['$scope', function($scope){
            $scope.customer = {
                name: 'Naomi',
                address: '1600 Amphitheatre'
            };
        }])
    .directive('myCustomer', function(){
            return {
                template: 'Name: {{customer.name}} Address: {{customer.address}}'
            }
        });

index.html

    <div ng-controller="Controller">
        <div my-customer></div>
    </div>    


注意这个指令的绑定，当`$compiler`编译并链接到`<div my-customer></div>`后，它将试着在子元素上匹配指令。这意味着，你可以在directive中嵌套其他directive。我们将看下面的列子是怎样做的。    

>最佳实践：除非模板非常的小，否则将它放入到一个单独的HTML文件中，使用`templateUrl`选项加载它。    

这有个同样的列子使用`templateUrl`来替换上面的列子：

script.js（注意directive方法，返回值属性的变化）

    .controller('Controller', ['$scope', function($scope){
            $scope.customer = {
                name: 'Naomi',
                address: '1600 Amphitheatre'
            };
        }])
    .directive('myCustomer', function(){
            return {
                templateUrl: 'template.html'
            }
        });

index.html

    <div ng-controller="Controller">
        <div my-customer></div>
    </div>    

template.html

    Name: {{customer.name}} Address: {{customer.address}}


如何使用`<my-customer>`表签名来匹配自定义指令？如果你将`<my-customer>`直接放入到HTML中，它是不会工作的。    

> 注意：当你创建一个指令的时候，默认是被限制在属性上的。如何让指令通过元素名或class来触发？你就需要使用`restrict`选项。

`restrict`选项有如下设置：

* 'A' - 只匹配属性名
* 'E' - 只匹配元素名
* 'C' - 只匹配class名

这些限制也可以连接在一起：

* 'AEC' - 即匹配属性名，也匹配元素名 和 class名    

让我们使用`restrict: 'E'`改变我们的指令：

script.js

    .controller('Controller', ['$scope', function($scope){
            $scope.customer = {
                name: 'Naomi',
                address: '1600 Amphitheatre'
            };
        }])
    .directive('myCustomer', function(){
            return {
                templateUrl: 'template.html'
            }
        });

index.html（注意标签名的变化）

    <div ng-controller="Controller">
        <my-customer></my-customer>
    </div>    

template.html

    Name: {{customer.name}} Address: {{customer.address}}


> 我们应该使用属性还是元素？使用元素创建指令是为了控制模板。这种情况是为了给模板创建“特定领域语言”。使用属性是为了给一个已存在的元素附加新的功能。    

    


##隔离指令作用域(Scope)

上面的列子有个致命的问题，我们只能在给的的作用域中使用一次。    
为了服用这个指令，我们必须每次创建不同的controller来使用这个指令。如下所示：    
script.js（注意controller写法，创建了两个controller）

    angular.module('docsScopeProblemExample', [])
      .controller('NaomiController', ['$scope', function($scope) {
        $scope.customer = {
          name: 'Naomi',
          address: '1600 Amphitheatre'
        };
      }])
      .controller('IgorController', ['$scope', function($scope) {
        $scope.customer = {
          name: 'Igor',
          address: '123 Somewhere'
        };
      }])
      .directive('myCustomer', function() {
        return {
          restrict: 'E',
          templateUrl: 'my-customer.html'
        };
      });


index.html

    <div ng-controller="NaomiController">
      <my-customer></my-customer>
    </div>
    <hr>
    <div ng-controller="IgorController">
      <my-customer></my-customer>
    </div>


template.html

    Name: {{customer.name}} Address: {{customer.address}}

这是个很聪明的方式，但不是最好的。    

我们希望能做到，指令的内部作用域与外部作用域相分离，然后映射外部作用域到内部作用域。我们可以使用`scope`选项达到这个目的，这被称之为`隔离作用域`。    

script.js(注意controller写法，只有一个；再者directive返回值增加了`scope`属性)

    angular.module('docsIsolateScopeDirective', [])
      .controller('Controller', ['$scope', function($scope) {
        $scope.naomi = { name: 'Naomi', address: '1600 Amphitheatre' };
        $scope.igor = { name: 'Igor', address: '123 Somewhere' };
      }])
      .directive('myCustomer', function() {
        return {
          restrict: 'E',
          scope: {
            customerInfo: '=info'
          },
          templateUrl: 'my-customer-iso.html'
        };
      });

index.html

    <div ng-controller="Controller">
      <my-customer info="naomi"></my-customer>
      <hr>
      <my-customer info="igor"></my-customer>
    </div>

注意index.html中第一个&lt;my-customer&gt;元素，info属性与naomi绑定，作用在controller作用域中。第二个元素的info属性与igor绑定。    

再看scope 选项：

    //...
    scope: {
        customerInfo: '=info'
    }
    //...

**scope**选项是一个对象，有一个属性绑定到隔离作用域，在这个例子中它只有一个属性：

* 它的名字(`customerInfo`)对应到指令隔离作用域的`customerInfo`属性。
* 它的值(`=info`)告诉$compiler去绑定`info`属性。

>注意：指令`scope`选项的`=attr`属性，通常被视为指令的名称。`<div bind-to-this="thing">`中的绑定属性，你必须指定为`=bindToThis`。

如果指令`scope`选项的属性名和属性值相同，则可以使用一个短语法： 

    ....
    scope: {
        // same as '=customer'
        customer: '='
    },
    .... 


> 注意：通常`scope`继承自父类，但是`isolate scope`不是。

> 最佳实践：如果想在你的应用中使用组件复用，可以使用`scope`选项创建 `isolate scope`。   


##创建一个可操作DOM的指令

我们将构建一个显示当前时间的列子，每2秒更新一次。    

使用`link`选项修改DOM，`link`接受一个函数签名，如：`function link(scope, element, attrs) { ... }`其中：   

* `scope` - 是一个Angular scope对象
* `element` - 是匹配指令的被jqLite包装的元素
* `attrs` - 以key/value键值对的形式，保存了元素对应的属性值

在`link`函数中我们每秒按照要求的格式更新一次时间。我们使用`$interval`服务定时处理。在指令删除时移除`$interval`，以防止内存耗尽。    

`module.directive`看起来和`module.controller` API很相似。因为要在指令的`link`函数中使用`$interval`和`dateFilter`，所以要为directive指定依赖关系。    

我们注册了一个`element.on('$destroy', ...)`事件。为什么要触发`$destroy`事件？    

AngularJS有几个特殊事件。当Angular compiler销毁被编译过的DOM节点时，会触发一个`$destroy`事件。相类似的，当Angular scope被销毁时，也会触发一个`$destroy`事件。    

通过监听这个事件，你可以移除引起内存耗尽的监听器。监听器注册到`scope`和`element`上时，当他们销毁时会被自动清除。但是如果一个监听器被注册到`service`、`DOM 节点`时是不会被自动删除的，因此你必须自己清除，否则会引起内存耗尽。    

> 最佳实践：你可以在指令移除时，通过`element.on('$destroy', ...)`和`scope.$on('$destroy', ...)`运行一个清除函数。    

    

##创建一个可包装其他元素的指令
我们通过使用`isloate scope`可以将模型传递到 directive中，但是有时候我们希望能将整个模板，而不是字符串或单个对象传入到 directive 中。我们想创建一个"dialog box"组件，这个组件可以包含任意内容。    

要做到这点，我们需要使用`transclude`选项。    

scripts.js（注意directive返回值，增加了transclude属性）

    angular.module('docsTransclustionDirective', [])
    .controller('Controller', ['$scope', function($scope){
            $scope.name = 'Tobias';
    }])
    .directive('myDialog', function(){
        return {
            restrict: 'E',
            transclude: true,
            templateUrl: 'my-dialog.html'
        }
    });


index.html

    <div ng-controller="Controller">
        <my-dialog>Check out the contents, {{name}}!</my-dialog>
    </div>

my-dialog.xml

    <div class="alert" ng-transclude>
    </div>


`transclude`做了什么？使用`transclude`选项可以使`directive`内的内容访问directive 之外的scope而不是之内的scope。    

为了说明这一点，可以看下面的列子。注意我们添加了一个`link`函数，在函数内我们重新定义了‘name’值为‘Jeff’。你认为这个绑定会被解析吗？


scripts.js（注意directive返回值，多增加了scope和link属性）

    angular.module('docsTransclustionDirective', [])
    .controller('Controller', ['$scope', function($scope){
            $scope.name = 'Tobias';
    }])
    .directive('myDialog', function(){
        return {
            restrict: 'E',
            transclude: true,
            scope: {},
            link: function(scope, element){
                scope.name = 'Jeff';
            },
            templateUrl: 'my-dialog.html'
        }
    });

我们期望{{name}}被替换为Jeff，但实际上它仍然是Tobias。    

`transclude`选项改变了scope的嵌套方式。它使的`transcluded directive`的内容可以访问 directive 外的scope，而不是directive 内的scope。    

>注意如果directive不创建自己的scope，则link函数内的操作是成立。因为这时后使用的是directive外的scope，我们可以看到Jeff输出。


> 最佳实践：当你想创建的指令可以包装任意内容时，可以使用`transclude: true`。    
    

下面，我们将向dialog box 中添加一个按钮。并允许别人使用这个directive绑定到他们自己的directive上。    

script.js

    angular.module('docsIsoFnBindExample', [])
    .controller('Controller', ['$scope', '$timeout', function($scope, $timeout){
        $scope.name = 'Tobias';
            $scope.hideDialog = function(){
                $scope.dialogIsHidden = true;
                $timeout(function(){
                    $scope.dialogIsHidden = false;
                }, 2000);
            }
    }])
    .directive('myDialogClose', function(){
        return {
            restrict: 'E',
            transclude: true,
            scope: {
                close: '&onClose'
            },
            templateUrl: 'my-dialog-close.html'
        }
    });

index.html

        <div ng-controller="Controller">
            <my-dialog-close ng-hide="dialogIsHidden" on-close="hideDialog()">
                Check out the contents, {{name}}!
            </my-dialog-close>
        </div>

my-dialog-close.html

    <div class="alert">
        <a href class="close" ng-click="close()">×</a>
        <div ng-transclude></div>
    </div>


在之前的例子中`scope`选项使用的是`=attr`，但在上面的列子中使用的是`&attr`。`&`允许directive在原始的scope上下文中触发表达式执行。任何合法的表达式都可以，包括表示函数调用的表达式。因此对于directive来说`&`表示函数回调。    


当点击dialog中的'x'时，directive的close 函数被调用。这个isolated scope中的close 的调用，实际是触发了原始scope上下文中的`hideDialog()`，然后运行`Controller`中定义的`hideDialog`函数。    


>最佳实践：当你想将你的directive暴露出一个API去绑定一个行为时，可以在scope中使用`&attr`。    


##创建一个带事件监听的指令
下面的列子演示了如何创建一个指令支持元素拖拽。    

script.js

    angular.module('docsDragModoul', [])
    .directive('myDraggable',['$document', function($document){
        return {
            link: function(scope, element, attr){
                var startX = 0, startY= 0, x= 0, y= 0;

                element.css({
                    position: 'relative',
                    border: '1px solid red',
                    backgroundColor: 'lightgrey',
                    cursor: 'pointer'
                });

                element.on('mousedown', function(event){
                    event.preventDefault();
                    startX = event.pageX - x;
                    startY = event.pageY - y;
                    $document.on('mousemove', mousemove);
                    $document.on('mouseup', mouseup);
                });

                function mousemove(event){
                    y = event.pageY - startY;
                    x = event.pageX - startX;
                    element.css({
                        top: y + 'px',
                        left:  x + 'px'
                    });
                }

                function mouseup(event){
                    $document.off('mousemove', mousemove);
                    $document.off('mouseup', mouseup);
                }
            }
        }
    }]);

index.html

    <span my-draggable>Drag ME</span>

##创建通讯指令

你可以在模板内使用任何指令。    

有时你希望通过组合directive来构建一个组件。    

下面我们来看看如果构建一个tabs组件。




##参考
https://docs.angularjs.org/guide/directive    










