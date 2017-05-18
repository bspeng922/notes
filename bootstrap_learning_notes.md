## Botstrap 基本样式

### 引用bootstrap
```html 
<!-- 新 Bootstrap 核心 CSS 文件 -->
<link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css">

<!-- 可选的Bootstrap主题文件（一般不用引入） -->
<link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap-theme.min.css">

<!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
<script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>

<!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
<script src="//cdn.bootcss.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
```

### 基本页面
Bootstrap 使用到的某些 HTML 元素和 CSS 属性需要将页面设置为 HTML5 文档类型。在你项目中的每个页面都要参照下面的格式进行设置。
```html 
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>Bootstrap 101 Template</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">

    <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="//cdn.bootcss.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="//cdn.bootcss.com/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <h1>你好，世界！</h1>

    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="js/bootstrap.min.js"></script>
  </body>
</html>
```

### 移动设备优先
Bootstrap 是移动设备优先的。针对移动设备的样式融合进了框架的每个角落，而不是增加一个额外的文件。为了确保适当的绘制和触屏缩放，需要在 <head\> 之中添加 viewport 元数据标签。
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```
在移动设备浏览器上，通过为视口（viewport）设置 meta 属性为 user-scalable=no 可以禁用其缩放（zooming）功能。这样禁用缩放功能后，用户只能滚动屏幕，就能让你的网站看上去更像原生应用的感觉。注意，这种方式我们并不推荐所有网站使用，还是要看你自己的情况而定！
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

### 全局样式
Bootstrap 排版、链接样式设置了基本的全局样式，这些样式都能在 scaffolding.less 文件中找到对应的源码
+ 为 body 元素设置 background-color: #fff;
+ 使用 @font-family-base、@font-size-base 和 @line-height-base a变量作为排版的基本参数
+ 为所有链接设置了基本颜色 @link-color ，并且当链接处于 :hover 状态时才添加下划线

### 布局容器
Bootstrap 需要为页面内容和栅格系统包裹一个 .container 容器。我们提供了两个作此用处的类。注意，由于 padding 等属性的原因，这两种 容器类不能互相嵌套。
+ .container 类用于固定宽度并支持响应式布局的容器。
+ .container-fluid 类用于 100% 宽度，占据全部视口（viewport）的容器。   
```html 
<div class="container">
  ...
</div>
<div class="container-fluid">
  ...
</div>
```

### 栅格系统
Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多12列。栅格系统用于通过一系列的行（row）与列（column）的组合来创建页面布局，你的内容就可以放入这些创建好的布局中。类似于表格布局

超小屏幕 手机 (<768px) | 小屏幕 平板 (≥768px) | 中等屏幕 桌面显示器 (≥992px) | 大屏幕 大桌面显示器 (≥1200px)
None （自动） | 750px | 970px | 1170px
.col-xs- | .col-sm- | .col-md- | .col-lg-

使用单一的一组 .col-md-* 栅格类，就可以创建一个基本的栅格系统，在手机和平板设备上一开始是堆叠在一起的（超小屏幕到小屏幕这一范围），在桌面（中等）屏幕设备上变为水平排列。所有“列（column）必须放在 ” .row 内。
```html 
<div class="row">
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
</div>
<div class="row">
  <div class="col-md-8">.col-md-8</div>
  <div class="col-md-4">.col-md-4</div>
</div>
<div class="row">
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4">.col-md-4</div>
</div>
<div class="row">
  <div class="col-md-6">.col-md-6</div>
  <div class="col-md-6">.col-md-6</div>
</div>
```


在某些阈值时，某些列可能会出现比别的列高的情况。为了克服这一问题，建议联合使用 .clearfix
```
<div class="row">
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>

  <!-- Add the extra clearfix for only the required viewport -->
  <div class="clearfix visible-xs-block"></div>

  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
</div>
```

除了列在分界点清除响应， 您可能需要 重置偏移, 后推或前拉某个列，例如，.col-md-offset-4 类将 .col-md-4 元素向右侧偏移了4个列（column）的宽度。
```
<div class="row">
  <div class="col-sm-5 col-md-6">.col-sm-5 .col-md-6</div>
  <div class="col-sm-5 col-sm-offset-2 col-md-6 col-md-offset-0">.col-sm-5 .col-sm-offset-2 .col-md-6 .col-md-offset-0</div>
</div>
```

### 嵌套列
为了使用内置的栅格系统将内容再次嵌套，可以通过添加一个新的 .row 元素和一系列 .col-sm-* 元素到已经存在的 .col-sm-* 元素内。被嵌套的行（row）所包含的列（column）的个数不能超过12（其实，没有要求你必须占满12列）。
```
<div class="row">
  <div class="col-sm-9">
    Level 1: .col-sm-9
    <div class="row">
      <div class="col-xs-8 col-sm-6">
        Level 2: .col-xs-8 .col-sm-6
      </div>
      <div class="col-xs-4 col-sm-6">
        Level 2: .col-xs-4 .col-sm-6
      </div>
    </div>
  </div>
</div>
```

### 列排序
通过使用 .col-md-push-* 和 .col-md-pull-* 类就可以很容易的改变列（column）的顺序
```
<div class="row">
  <div class="col-md-9 col-md-push-3">.col-md-9 .col-md-push-3</div>
  <div class="col-md-3 col-md-pull-9">.col-md-3 .col-md-pull-9</div>
</div>
```

### 排版
HTML 中的所有标题标签，<h1\> 到 <h6\> 均可使用，在标题内还可以包含 <small\> 标签或赋予 .small 类的元素，可以用来标记副标题。
Bootstrap 将全局 font-size 设置为 14px，line-height 设置为 1.428。这些属性直接赋予 <body> 元素和所有段落元素。另外，<p\> （段落）元素还被设置了等于 1/2 行高（即 10px）的底部外边距（margin）。

#### 中心内容
通过添加 .lead 类可以让段落突出显示。
```html 
<p class="lead">...</p>
```

#### 内联元素
+ 高亮文本
For highlighting a run of text due to its relevance in another context, use the <mark\> tag.
+ 被删除的文本
对于被删除的文本使用 <del\> 标签。
+ 无用文本
对于没用的文本使用 <s\> 标签。
+ 插入文本
额外插入的文本使用 <ins\> 标签。
+ 带下划线的文本
为文本添加下划线，使用 <u\> 标签。
+ 小号文本
对于不需要强调的inline或block类型的文本，使用 <small\> 标签包裹，其内的文本将被设置为父容器字体大小的 85%。标题元素中嵌套的 <small\> 元素被设置不同的 font-size 。
你还可以为行内元素赋予 .small 类以代替任何 <small\> 元素。
+ 着重
通过增加 font-weight 值强调一段文本， 使用<strong\>标签
+ 斜体
用斜体强调一段文本， 使用<em\> 元素

> 在 HTML5 中可以放心使用 <b\> 和 <i\> 标签。<b\> 用于高亮单词或短语，不带有任何着重的意味；而 <i\> 标签主要用于发言、技术词汇等。

+ 对齐
通过文本对齐类，可以简单方便的将文字重新对齐
```html 
<p class="text-left">Left aligned text.</p>
<p class="text-center">Center aligned text.</p>
<p class="text-right">Right aligned text.</p>
<p class="text-justify">Justified text.</p>
<p class="text-nowrap">No wrap text.</p>
```

+ 改变大小写
通过这几个类可以改变文本的大小写
```html 
<p class="text-lowercase">Lowercased text.</p>
<p class="text-uppercase">Uppercased text.</p>
<p class="text-capitalize">Capitalized text.</p>
```

+ 缩略语
当鼠标悬停在缩写和缩写词上时就会显示完整内容，Bootstrap 实现了对 HTML 的 <abbr\> 元素的增强样式。缩略语元素带有 title 属性，外观表现为带有较浅的虚线框，鼠标移至上面时会变成带有“问号”的指针。如想看完整的内容可把鼠标悬停在缩略语上（对使用辅助技术的用户也可见）, 但需要包含 title 属性。

为缩略语添加 .initialism 类，可以让 font-size 变得稍微小些。
```html 
<abbr title="HyperText Markup Language" class="initialism">HTML</abbr>
```

+ 地址 
让联系信息以最接近日常使用的格式呈现。在每行结尾添加 <br\> 可以保留需要的样式。
```html 
<address>
  <strong>Twitter, Inc.</strong><br>
  795 Folsom Ave, Suite 600<br>
  San Francisco, CA 94107<br>
  <abbr title="Phone">P:</abbr> (123) 456-7890
</address>
```

+ 引用
将任何 HTML 元素包裹在 <blockquote\> 中即可表现为引用样式。对于直接引用，我们建议用 <p\> 标签。
添加 <footer\> 用于标明引用来源。来源的名称可以包裹进 <cite\>标签中。
通过赋予 .blockquote-reverse 类可以让引用呈现内容右对齐的效果。
```html 
<blockquote>
  <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Integer posuere erat a ante.</p>
  <footer>Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>
```

+ 列表
无序列表
```
<ul>
  <li>...</li>
</ul>
```
有序列表
```
<ol>
  <li>...</li>
</ol>
```
移除了默认的 list-style 样式和左侧外边距的一组元素（只针对直接子元素）。这是针对直接子元素的，也就是说，你需要对所有嵌套的列表都添加这个类才能具有同样的样式。

通过设置 display: inline-block; 并添加少量的内补（padding），将所有元素放置于同一行。

+ 描述
```
<dl>
  <dt>...</dt>
  <dd>...</dd>
</dl>
```
.dl-horizontal 可以让 <dl> 内的短语及其描述排在一行。开始是像 <dl\> 的默认样式堆叠在一起，随着导航条逐渐展开而排列在一行。

> 通过 text-overflow 属性，水平排列的描述列表将会截断左侧太长的短语。在较窄的视口（viewport）内，列表将变为默认堆叠排列的布局方式。通过 text-overflow 属性，水平排列的描述列表将会截断左侧太长的短语。在较窄的视口（viewport）内，列表将变为默认堆叠排列的布局方式。


### 代码
通过 <code\> 标签包裹内联样式的代码片段
通过 <kbd\> 标签标记用户通过键盘输入的内容
```
To edit settings, press <kbd><kbd>ctrl</kbd> + <kbd>,</kbd></kbd>
```

多行代码可以使用 <pre\> 标签。为了正确的展示代码，注意将尖括号做转义处理。
还可以使用 .pre-scrollable 类，其作用是设置 max-height 为 350px ，并在垂直方向展示滚动条。

通过 <var\> 标签标记变量。
通过 <samp\> 标签来标记程序输出的内容


### 表格
为任意 <table\> 标签添加 .table 类可以为其赋予基本的样式 — 少量的内补（padding）和水平方向的分隔线。
通过 .table-striped 类可以给 <tbody\> 之内的每一行增加斑马条纹样式。
```html 
<table class="table table-striped">
  ...
</table>
```

添加 .table-bordered 类为表格和其中的每个单元格增加边框。
通过添加 .table-hover 类可以让 <tbody\> 中的每一行对鼠标悬停状态作出响应。
通过添加 .table-condensed 类可以让表格更加紧凑，单元格中的内补（padding）均会减半。

通过这些状态类可以为行或单元格设置颜色
.active | 鼠标悬停在行或单元格上时所设置的颜色
.success  |  标识成功或积极的动作
.info  | 标识普通的提示信息或动作
.warning  |  标识警告或需要用户注意
.danger | 标识危险或潜在的带来负面影响的动作
```html 
<!-- On rows -->
<tr class="active">...</tr>
<tr class="success">...</tr>

<!-- On cells (`td` or `th`) -->
<tr>
  <td class="active">...</td>
  <td class="success">...</td>
</tr>
```

将任何 .table 元素包裹在 .table-responsive 元素内，即可创建响应式表格，其会在小屏幕设备上（小于768px）水平滚动。当屏幕大于 768px 宽度时，水平滚动条消失。
```html 
<div class="table-responsive">
  <table class="table">
    ...
  </table>
</div>
```

### 表单
单独的表单控件会被自动赋予一些全局样式。所有设置了 .form-control 类的 <input\>、<textarea\> 和 <select\> 元素都将被默认设置宽度属性为 width: 100%;。 将 label 元素和前面提到的控件包裹在 .form-group 中可以获得最好的排列。
```html 
<form>
  <div class="form-group">
    <label for="exampleInputEmail1">Email address</label>
    <input type="email" class="form-control" id="exampleInputEmail1" placeholder="Email">
  </div>
  <div class="form-group">
    <label class="sr-only" for="exampleInputEmail3">Email address</label>
    <input type="email" class="form-control" id="exampleInputEmail3" placeholder="Email">
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
</form>
```

> 不要将表单组直接和输入框组混合使用。建议将输入框组嵌套到表单组中使用。
> 一定要添加 label 标签, 如果你没有为每个输入控件设置 label 标签，屏幕阅读器将无法正确识别。对于这些内联表单，你可以通过为 label 设置 .sr-only 类将其隐藏。

为 <form\> 元素添加 .form-inline 类可使其内容左对齐并且表现为 inline-block 级别的控件。只适用于视口（viewport）至少在 768px 宽度时（视口宽度再小的话就会使表单折叠）。

+ 输入框
如需在文本输入域 <input\> 前面或后面添加文本内容或按钮控件,可以添加.input-group,通过.input-group-addon来为输入框添加一些额外的信息
```html 
<form class="form-inline">
  <div class="form-group">
    <label class="sr-only" for="exampleInputAmount">Amount (in dollars)</label>
    <div class="input-group">
      <div class="input-group-addon">$</div>
      <input type="text" class="form-control" id="exampleInputAmount" placeholder="Amount">
      <div class="input-group-addon">.00</div>
    </div>
  </div>
  <button type="submit" class="btn btn-primary">Transfer cash</button>
</form>
```


通过为表单添加 .form-horizontal 类，并联合使用 Bootstrap 预置的栅格类，可以将 label 标签和控件组水平并排布局。这样做将改变 .form-group 的行为，使其表现为栅格系统中的行（row），因此就无需再额外添加 .row 了。
```html 
<form class="form-horizontal">
  <div class="form-group">
    <label for="inputEmail3" class="col-sm-2 control-label">Email</label>
    <div class="col-sm-10">
      <input type="email" class="form-control" id="inputEmail3" placeholder="Email">
    </div>
  </div>
</form>
```

bootstrap包括大部分表单控件、文本输入域控件，还支持所有 HTML5 类型的输入控件： text、password、datetime、datetime-local、date、month、time、week、number、email、url、search、tel 和 color。
> 只有正确设置了 type 属性的输入控件才能被赋予正确的样式。

+ 多选和单选框
设置了 disabled 属性的单选或多选框都能被赋予合适的样式。对于和多选或单选框联合使用的 <label\> 标签，如果也希望将悬停于上方的鼠标设置为“禁止点击”的样式，请将 .disabled 类赋予 .radio、.radio-inline、.checkbox、.checkbox-inline 或 <fieldset\>。
```html 
<div class="radio disabled">
  <label>
    <input type="radio" name="optionsRadios" id="optionsRadios3" value="option3" disabled>
    Option three is disabled
  </label>
</div>
```

通过将 .checkbox-inline 或 .radio-inline 类应用到一系列的多选框（checkbox）或单选框（radio）控件上，可以使这些控件排列在一行。

如果需要 <label\> 内没有文字，输入框（input）正是你说期望的。 目前只适用于非内联的 checkbox 和 radio。 请记住，仍然需要为使用辅助技术的用户提供某种形式的 label（例如，使用 aria-label）。
```
<div class="checkbox">
  <label>
    <input type="checkbox" id="blankCheckbox" value="option1" aria-label="...">
  </label>
</div>
```

+ 下拉列表
很多原生选择菜单 - 即在 Safari 和 Chrome 中 - 的圆角是无法通过修改 border-radius 属性来改变的
对于标记了 multiple 属性的 <select\> 控件来说，默认显示多选项

+ 静态控件
如果需要在表单中将一行纯文本和 label 元素放置于同一行，为 <p\> 元素添加 .form-control-static 类即可。
```
<div class="form-group">
    <label class="col-sm-2 control-label">Email</label>
    <div class="col-sm-10">
      <p class="form-control-static">email@example.com</p>
    </div>
  </div>
```

为输入框设置 disabled 属性可以禁止其与用户有任何交互（焦点、输入等）。被禁用的输入框颜色更浅，并且还添加了 not-allowed 鼠标状态。

为<fieldset\> 设置 disabled 属性,可以禁用 <fieldset\> 中包含的所有控件。<a\> 标签的链接功能不受影响

为输入框设置 readonly 属性可以禁止用户修改输入框中的内容。处于只读状态的输入框颜色更浅（就像被禁用的输入框一样），但是仍然保留标准的鼠标状态。

Bootstrap 对表单控件的校验状态，如 error、warning 和 success 状态，都定义了样式。使用时，添加 .has-warning、.has-error 或 .has-success 类到这些控件的父元素即可。任何包含在此元素之内的 .control-label、.form-control 和 .help-block 元素都将接受这些校验状态的样式。
```
<div class="has-success">
  <div class="checkbox">
    <label>
      <input type="checkbox" id="checkboxSuccess" value="option1">
      Checkbox with success
    </label>
  </div>
</div>
```

你还可以针对校验状态为输入框添加额外的图标。只需设置相应的 .has-feedback 类并添加正确的图标即可,反馈图标（feedback icon）只能使用在文本输入框 <input class="form-control"\> 元素上。
```
<div class="form-group has-success has-feedback">
  <label class="control-label" for="inputGroupSuccess1">Input group with success</label>
  <div class="input-group">
    <span class="input-group-addon">@</span>
    <input type="text" class="form-control" id="inputGroupSuccess1" aria-describedby="inputGroupSuccess1Status">
  </div>
  <span class="glyphicon glyphicon-ok form-control-feedback" aria-hidden="true"></span>
  <span id="inputGroupSuccess1Status" class="sr-only">(success)</span>
</div>
```

如果你使用 .sr-only 类来隐藏表单控件的 <label\> （而不是使用其它标签选项，如 aria-label 属性）， 一旦它被添加，Bootstrap 会自动调整图标的位置。

通过 .input-lg 类似的类可以为控件设置高度，通过 .col-lg-* 类似的类可以为控件设置宽度。

通过添加 .form-group-lg 或 .form-group-sm 类，为 .form-horizontal 包裹的 label 元素和表单控件快速设置尺寸。

用栅格系统中的列（column）包裹输入框或其任何父元素，都可很容易的为其设置宽度。

与表单控件相关联的帮助文本 aria-describedby 属性的表单控件关联，这将确保使用辅助技术- 如屏幕阅读器 - 的用户获取控件焦点或进入控制时显示这个帮助文本。
```
<label class="sr-only" for="inputHelpBlock">Input with help text</label>
<input type="text" id="inputHelpBlock" class="form-control" aria-describedby="helpBlock">
...
<span id="helpBlock" class="help-block">A block of help text that breaks onto a new line and may extend beyond one line.</span>
```


### 按钮
为 <a\>、<button\> 或 <input\> 元素添加按钮类（button class）即可使用 Bootstrap 提供的样式。
虽然按钮类可以应用到 <a\> 和 <button\> 元素上，但是，导航和导航条组件只支持 <button\> 元素。

如果 <a\> 元素被作为按钮使用 -- 并用于在当前页面触发某些功能 -- 而不是用于链接其他页面或链接当前页面中的其他部分，那么，务必为其设置 role="button" 属性。

我们总结的最佳实践是：强烈建议尽可能使用 <button\> 元素来获得在各个浏览器上获得相匹配的绘制效果
使用 .btn-lg、.btn-sm 或 .btn-xs 就可以获得不同尺寸的按钮
通过给按钮添加 .btn-block 类可以将其拉伸至父元素100%的宽度，而且按钮也变为了块级（block）元素

```
<!-- Standard button -->
<button type="button" class="btn btn-default">（默认样式）Default</button>

<!-- Provides extra visual weight and identifies the primary action in a set of buttons -->
<button type="button" class="btn btn-primary">（首选项）Primary</button>

<!-- Indicates a successful or positive action -->
<button type="button" class="btn btn-success">（成功）Success</button>

<!-- Contextual button for informational alert messages -->
<button type="button" class="btn btn-info">（一般信息）Info</button>

<!-- Indicates caution should be taken with this action -->
<button type="button" class="btn btn-warning">（警告）Warning</button>

<!-- Indicates a dangerous or potentially negative action -->
<button type="button" class="btn btn-danger">（危险）Danger</button>

<!-- Deemphasize a button by making it look like a link while maintaining button behavior -->
<button type="button" class="btn btn-link">（链接）Link</button>
```

当按钮处于激活状态时，其表现为被按压下去（底色更深、边框夜色更深、向内投射阴影）。对于 <button\> 元素，是通过 :active 状态实现的。对于 <a\> 元素，是通过 .active 类实现的。然而，你还可以将 .active 应用到 <button\> 上（包含 aria-pressed="true" 属性)），并通过编程的方式使其处于激活状态。
由于 :active 是伪状态，因此无需额外添加，但是在需要让其表现出同样外观的时候可以添加 .active 类。

通过为按钮的背景设置 opacity 属性就可以呈现出无法点击的效果。

为 <button\> 元素添加 disabled 属性，使其表现出禁用状态。为基于 <a\> 元素创建的按钮添加 .disabled 类。

### 图片
在 Bootstrap 版本 3 中，通过为图片添加 .img-responsive 类可以让图片支持响应式布局。其实质是为图片设置了 max-width: 100%;、 height: auto; 和 display: block; 属性，从而让图片在其父元素中更好的缩放。
如果需要让使用了 .img-responsive 类的图片水平居中，请使用 .center-block 类，不要用 .text-center。 

通过为 <img\> 元素添加以下相应的类，可以让图片呈现不同的形状，需要支持 CSS3
```
<img src="..." alt="..." class="img-rounded">
<img src="..." alt="..." class="img-circle">
<img src="..." alt="..." class="img-thumbnail">
```


### 辅助类
+ 情境颜色
通过颜色来展示意图，Bootstrap 提供了一组工具类。这些类可以应用于链接，并且在鼠标经过时颜色可以还可以加深，就像默认的链接一样。
和情境文本颜色类一样，使用任意情境背景色类就可以设置元素的背景。链接组件在鼠标经过时颜色会加深，就像上面所讲的情境文本颜色类一样。
```
<p class="text-muted">...</p>
<p class="text-primary">...</p>
<p class="text-success">...</p>
<p class="text-info">...</p>
<p class="text-warning">...</p>
<p class="text-danger">...</p>

<p class="bg-primary">...</p>
<p class="bg-success">...</p>
<p class="bg-info">...</p>
<p class="bg-warning">...</p>
<p class="bg-danger">...</p>
```

+ 关闭按钮
```
<button type="button" class="close" aria-label="Close"><span aria-hidden="true">&times;</span></button>
```
+ 三角符号
```
<span class="caret"></span>
```
+ 快速浮动
排列导航条中的组件时可以使用这些工具类：.navbar-left 或 .navbar-right 
```
<div class="pull-left">...</div>
<div class="pull-right">...</div>
```
+ 让内容块居中
为任意元素设置 display: block 属性并通过 margin 属性让其中的内容居中
```
<div class="center-block">...</div>
```
+ 清除浮动
通过为父元素添加 .clearfix 类可以很容易地清除浮动（float）。
```
<div class="clearfix">...</div>
```
+ 显示或隐藏内容
.show 和 .hidden 类可以强制任意元素显示或隐藏(对于屏幕阅读器也能起效)。这些类通过 !important 来避免 CSS 样式优先级问题，就像 quick floats 一样的做法。注意，这些类只对块级元素起作用

.sr-only 类可以对屏幕阅读器以外的设备隐藏内容，.sr-only 和 .sr-only-focusable 联合使用的话可以在元素有焦点的时候再次显示出来（例如，使用键盘导航的用户）

+ 图片替换
使用 .text-hide 类或对应的 mixin 可以用来将元素的文本内容替换为一张背景图。

颜色
```
@gray-darker:  lighten(#000, 13.5%); // #222
@gray-dark:    lighten(#000, 20%);   // #333
@gray:         lighten(#000, 33.5%); // #555
@gray-light:   lighten(#000, 46.7%); // #777
@gray-lighter: lighten(#000, 93.5%); // #eee

@brand-primary: darken(#428bca, 6.5%); // #337ab7
@brand-success: #5cb85c;
@brand-info:    #5bc0de;
@brand-warning: #f0ad4e;
@brand-danger:  #d9534f;
```

### modal传数据
```html
<button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal" data-whatever="@mdo">Open modal for @mdo</button>
<button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal" data-whatever="@fat">Open modal for @fat</button>
<button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal" data-whatever="@getbootstrap">Open modal for @getbootstrap</button>
...more buttons...

<div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title" id="exampleModalLabel">New message</h4>
      </div>
      <div class="modal-body">
        <form>
          <div class="form-group">
            <label for="recipient-name" class="control-label">Recipient:</label>
            <input type="text" class="form-control" id="recipient-name">
          </div>
          <div class="form-group">
            <label for="message-text" class="control-label">Message:</label>
            <textarea class="form-control" id="message-text"></textarea>
          </div>
        </form>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Send message</button>
      </div>
    </div>
  </div>
</div>
```

```js
$('#exampleModal').on('show.bs.modal', function (event) {
  var button = $(event.relatedTarget) // Button that triggered the modal
  var recipient = button.data('whatever') // Extract info from data-* attributes
  // If necessary, you could initiate an AJAX request here (and then do the updating in a callback).
  // Update the modal's content. We'll use jQuery here, but you could use a data binding library or other methods instead.
  var modal = $(this)
  modal.find('.modal-title').text('New message to ' + recipient)
  modal.find('.modal-body input').val(recipient)
})
```
通过 data 属性或 JavaScript 调用模态框插件，可以根据需要动态展示隐藏的内容。模态框弹出时还会为 <body\> 元素添加 .modal-open 类，从而覆盖页面默认的滚动行为，并且还会自动生成一个 .modal-backdrop 元素用于提供一个可点击的区域，点击此区域就即可关闭模态框。

不需写 JavaScript 代码也可激活模态框。通过在一个起控制器作用的元素（例如：按钮）上添加 data-toggle="modal" 属性，或者 data-target="#foo" 属性，再或者 href="#foo" 属性，用于指向被控制的模态框。
```
$('#myModal').modal({
  keyboard: false
})
$('#myModal').modal('toggle')
$('#myModal').modal('show')
$('#myModal').modal('hide')
$('#myModal').modal('handleUpdate')
```
show.bs.modal  | show 方法调用之后立即触发该事件。如果是通过点击某个作为触发器的元素，则此元素可以通过事件的 relatedTarget 属性进行访问。
shown.bs.modal | 此事件在模态框已经显示出来（并且同时在 CSS 过渡效果完成）之后被触发。如果是通过点击某个作为触发器的元素，则此元素可以通过事件的 relatedTarget 属性进行访问。
hide.bs.modal  | hide 方法调用之后立即触发该事件。
hidden.bs.modal | 此事件在模态框被隐藏（并且同时在 CSS 过渡效果完成）之后被触发。
loaded.bs.modal | 从远端的数据源加载完数据之后触发该事件。

$('#myModal').on('hidden.bs.modal', function (e) {
  // do something...
})

### 下拉菜单
Add data-toggle="dropdown" to a link or button to toggle a dropdown.
```
<div class="dropdown">
  <a id="dLabel" data-target="#" href="http://example.com" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">
    Dropdown trigger
    <span class="caret"></span>
  </a>

  <ul class="dropdown-menu" aria-labelledby="dLabel">
    ...
  </ul>
</div>
```

### 滚动监听
滚动监听插件依赖 Bootstrap 的导航组件 用于高亮显示当前激活的链接
无论何种实现方式，滚动监听都需要被监听的组件是 position: relative; 即相对定位方式。大多数时候是监听 <body\> 元素
```
body {
  position: relative;
}

<body data-spy="scroll" data-target="#navbar-example">
  ...
  <div id="navbar-example">
    <ul class="nav nav-tabs" role="tablist">
      ...
    </ul>
  </div>
  ...
</body>

$('body').scrollspy({ target: '#navbar-example' })
```
当使用滚动监听插件的同时在 DOM 中添加或删除元素后，你需要像下面这样调用此刷新（ refresh） 方法
```
$('[data-spy="scroll"]').each(function () {
  var $spy = $(this).scrollspy('refresh')
})

activate.bs.scrollspy |  每当一个新条目被激活后都将由滚动监听插件触发此事件。
```
