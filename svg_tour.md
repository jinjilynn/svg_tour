# About SVG

可缩放矢量图,一种XML应用

# 图形系统

- ## 栅格图形

  图形是由单个像素拼凑而成,存在形式为jpg、png等二进制文件,可以进行修改但本身不具有编程性

- ## 矢量图形

  图形被描述为一系列的指令,以XML文件形式存在,具有可编程性

# SVG文档结构

- 声名  告知解析器使用xml处理指令

  ```xml
  <?xml version="1.0" ?>
  ```

- 文档定义类型 定义xml文档编写规范

  ```xml
  <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
  ```

- 根元素

  ```xml
  <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" >
  ```

## 在HTML中引用SVG

### 不能与主页面交互

- img标签
- css样式

以上两种引入方式中,svg最终会被转化为栅格图片显示,所以如果svg内部定义的外链脚本也将会无法被加载到,

只是与引入栅格图片不同的是,svg的固有大小计算略微复杂并且不固定:

- svg有明确width和height属性的,则固有大小尺寸按明确的width和height计算
- svg没有width和width属性但有viewbox属性的,则固有尺寸按viewbox定义的width和height计算
- svg只有width或height属性中的一个,并有viewbox属性的,则固有尺寸按width或height中的一个计算,并按viewbox定义的比例缩放
- 其他情况则按父元素与自身内容共同决定

### 可以与主页面交互

- object标签
- 内联SVG

内联svg可以省略声明和文档类型定义部分

## 在SVG中引用HTML

- foreignObject

里面的父Html标签要声明xhtml命名空间

## viewport

svg画布无限大,视口就是svg文件的固有尺寸大小,也就是svg的可见尺寸大小,可以理解成\<canvas\>标签的尺寸大小,视口大小的单位可以是绝对单位如cm或相对单位em.不论相对单位还是绝对单位都与当前系统的像素单位存在一定的换算关系.

###  viewport坐标系

一个视口确定好后,解析器会为其定义一个笛卡尔坐标系,视口左上角为原点,x轴向右正向递增,y轴向下正向递增,这个坐标系也就是svg标签所使用的坐标系.

在svg中所有子元素都会使用父元素的坐标系,所以在svg中的所有子元素都会使用这个默认的视口坐标系.但这种“使用”不是“引用”,而是一种从父到子、从上到下的同步、继承机制,也就是说其实每个绘图指令都有自己的坐标系,元素和元素之间的坐标系是相互独立的,只是当前绘图指令所使用的坐标系会继承父元素坐标系的变换,但是当前绘图指令的坐标系改变不会反向影响到父元素的坐标系.

### 坐标系变换

坐标系变换可以应用到父元素以使所有子元素使用到的坐标系都跟着变换,也可以单独应用到某个子元素只使它自己的坐标系变换.不论哪种变换最终影响到的都是绘图指令所使用的点的坐标.

所有的变换类型都写在元素的transform属性中,不能写在style属性中

#### 变换类型

- translate(x-value y-value)

  平移就是将当前坐标系相对于父元素坐标系上下左右移动到指定的单位距离,所以当前坐标系统下的所有图形也会相对于父元素的坐标系统发生相应的位移.

  语法:使用逗号或空格分隔,但不能有单位,单位为视口尺寸定义的单位.y-value为可选值,如果省略则为0.

- scale(x-value y-value)

  缩放是对当前坐标系的单位长度相对于父元素的坐标系进行倍数的缩放,原来父元素一个单位可能是指1px,如果对当前坐标系使用scale(2)进行变换,那么最终结果就是当前坐标系中一个单位等于2px,图形在视觉上看起来就会离原点更远且更长.

  语法: 使用逗号或空格分隔,.y-value为可选值,如果省略则为x-value.

- rotate(angle centerX  centerY)

  旋转就是当前坐标系相对于父元素坐标系以哪个点进行旋转以及旋转了多少度,规定顺时针为正向角度

  语法: angle为旋转角度 centerX centerY可以省略,为旋转中心,只能是绝对坐标

- skewX(angle) | skewY(angle)

  倾斜就是按指定角度推动坐标轴, 同样是相对于父元素坐标系.skewX推动的是y轴,skewY推动的是x轴

#### 变换组合

变换组合就是一个transform属性中,可以使用空格或逗号分隔多个变换类型,这里变换类型的出现就有个先后顺序的问题,在svg中多个变换类型组合在一起就相当于对当前元素嵌套了多层父元素,每一层父元素应用了相应的变换.

例如有这么一个长方形:

```xml
<rect transform="translate(20 20) scale()1.5" x="10" y="10" width="20" height="30" fill="black" />
```

这个长方形的变换组合就相当于

```xml
<g transform="translate(20 20)">
  <g transform="scale(1.5)">
  	<rect x="10" y="10" width="20" height="30" fill="black" />
	</g>
</g>
```



## viewbox

viewBox视窗是定义在svg标签上的一个属性,它语法是viewBox="x-value y-value width-value height-value".值与值之间可以用空格或逗号分隔

有的文章讲viewBox用了大量篇幅但是仍然讲不明白,我一点很奇怪.其实viewBox就干了一件事情:  <u>**先截取后平铺**</u>

x-value是截取矩形左上角的x坐标

y-value是截取矩形左上角的y坐标

width-value是截取矩形的宽度

height-value是截取矩形的高度

截取完成后再把截取的矩形图像平铺到视口中,这里的平铺是截取矩形的宽高比和视口的宽高比一致情况下的平铺

这种效果完全可以使用transform变换得到,就是先按 viewport-width / width-value、viewport-height / height-value的比例缩放坐标系,然后再移动坐标系使截取矩形的左上角与视口的左上角重合 translate(-[x-value] -[y-value])

比如说这么一段代码

```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg t="1607696353790" class="icon"  version="1.1" preserveAspectRatio="xMinyMax meet"
    xmlns="http://www.w3.org/2000/svg" p-id="3351"
    xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="600">
    <defs>
        <style type="text/css"></style>
    </defs>

    <rect x="300" y ="400" width="500" height="200" fill="black" />
</svg>
```

加上一个viewBox属性

```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg t="1607696353790" class="icon" viewBox="150,150,1200,1200"  version="1.1" preserveAspectRatio="xMinyMax meet"
    xmlns="http://www.w3.org/2000/svg" p-id="3351"
    xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="600">
    <defs>
        <style type="text/css"></style>
    </defs>

    <rect x="300" y ="400" width="500" height="200" fill="black" />
</svg>
```

加一个transform属性

```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg t="1607696353790" class="icon"  version="1.1" preserveAspectRatio="xMinyMax meet"
    xmlns="http://www.w3.org/2000/svg" p-id="3351"
    xmlns:xlink="http://www.w3.org/1999/xlink" width="600" height="600">
    <defs>
        <style type="text/css"></style>
    </defs>
    <g transform="scale(.5) translate(-150 -150)">
        <rect x="300" y ="400" width="500" height="200" fill="black" />
    </g>
</svg>
```

加viewBox与加transform效果相同

### preserveAspectRatio

以上仅限于viewbox的宽高比与viewport的宽高比相同的情况下,在viewbox的宽高比与viewport的宽高比不同的情况下有两种处理方法:

- 保留viewbox宽高比
- 不保留viewbox宽高比

#### 保留viewbox宽高比

保留viewbox的宽高比又分两种情况

- meet

  meet就相当于css样式background-size取值contain, 图像完全包含在viewport中,在viewport中存在定位问题

- slice

  slice就相当于css样式background-size取值cover,图像完全覆盖viewport,在viewport中存在截取问题

不论定位还是截取都存在对齐方式的问题,对齐方式就是xMin xMid xMax与yMin yMid yMax的组合

语法:preserveAspectRatio="alignment [meet | slice]"

#### 不保留viewbox宽高比

就是将图像完全平铺在viewport中,因为比例不同所以会产生变形.

语法: preserveAspectRatio="none"

## 常用指令

有了坐标系之后就可以开始图形的绘制工作了,在svg中绘制图形遵循的原则是表现与形状分离,所以样式指令在不同的形状中基本上是通用的.就像HTML与CSS一样,SVG中的形状指令是以元素标签的形式存在的,而样式指令则会作为元素标签属性的一部分以字符的形式存在.与HTML不同的是,SVG标签的绘制需要有精准坐的标定位,所以具有极强的可编程性.

### 形状指令

#### line

- \<line x1="x-value" y1="y-value" x2="x2-value" y2="y2-value" /\>

  line标签指令是画一条单一的线段路径,它特有的属性x1、y1、x2、y2份别值直线的起点和终点坐标

  value值可以有单位也可以没有单位

#### rect

- \<rect x="x-value" y="y-value" rx="rx-value" ry="ry-value" width="w-value" height="h-value" \>

  rect标签指令是画一条矩形路径,它特有的属性是x、y、width、height、rx、ry,其中x、y表示矩形左上角的顶点坐标;width、height表示矩形的长度和宽度;rx、ry表示在x方向和y方向上的圆角半径,最大不能超过各个方向上的一半,和css的border-radius样式不同,rx和ry代表了四个角的圆角半径,不能单独设置每个角的圆角半径,这两个是可选填参数,不是必须的.

  value值可以有单位也可以没有单位

#### circle

- \<circle cx="x-value" cy="y-value" r="r-value" \>

  circle标签指令是画一条圆弧路径,它特有的属性是cx、cy和r,可能是为了使属性名更具语义化所以为圆心坐标起了个cx和cy的名称,r自然就是半径

  value值可以有单位也可以没有单位

#### ellipse

- \<ellipse cx="x-value" cy="y-value" rx="rx-value" ry="ry-value" \>

  ellipse标签指令是画一条椭圆路径cx、cy表示椭圆的圆心坐标,,rx、ry表示在x方向和y方向上的椭圆半径.

  这个椭圆路径是完全可以使用rect路径画出来的,ellipse只是看起来更语义化一点

  value值可以有单位也可以没有单位

#### polyline

- \<polyline points="points-list" \>

  polyline标签指令表示一条连续的多条线段组成的路径,points是一系列的坐标对,坐标对与坐标对之间可以使用空格或逗号分隔,坐标对内的坐标值之间也可以使用空格或逗号分隔,总之可以随意分隔,但没有规律的分隔可读性就会非常差

#### polygon

- \<polygon points="points-list" \>

  polygon标签指令也表示一条连续的多条线段组成的路径,但是和polyline有一点区别就是起始点和终点也会连接上线段,而polyline就不会连接起点和终点坐标.

### 结构指令

结构指令与形状指令不同,它不会产生形状路径,对页面的视觉表现没有直接的影响,只是起到分组、复用等作用,使svg文档结构更具合理性.

#### g

g是group的缩写,可以在逻辑上使所有的子元素作为一组,同时为g标签定义的样式都是被所有子元素继承

#### use

use标签提供了复制粘贴使用元素的功能,被复制的元素可以是任何页面里出现的元素

```xml
<use xlink:href="" x="" y="" [style="" width="" height="" ...] />
```

xlink:href里定义任何可以定位到某个元素的url,虽然可能会因为同源策略被限制,因浏览器而已,一般指定一个id即可.

由于被复制的元素也是使用坐标点严格绘制出来的,所以这里的x、y所起到的作用就相当于为被复制元素应用translate坐标变换.

同时可以定义其他属性,比如style、width、height等

如果设置了width、height,相当于指定了可视部分的窗口大小,如果href引用了symbol等可以设置viewBox的元素,则会进行preserveAspectRatio适应

style里的样式会被复制元素继承. 

#### defs

在defs中定义的标签不会在页面中显示,可以用来定义后续需要被复用的标签或内部样式表style标签.

#### symbol

symbol和g标签一样都可以起到分组的作用,但是在symbol中的元素是永远不会显示出来的,这和定义在defs中的元素一样,都是供后续复用使用的元素.而在复用的时候和g标签又是有区别的,symbol可以定义viewBox和preserveAspectRatio,这样就会适配use表签定义的width和height属性.

#### image

image标签相当于建立起了一个视口,这个视口是基于定义的x、y、width、height属性来建立,可以为通过xlink:href属性引用外部完整的svg或栅格图片文件.

如果引入的图像文件宽高和image标签定义的宽高不匹配,则可以定义preserveAspectRatio属性来指定缩放.

```xml
<image x="" y ="" width="" height="" xlink:href="" preserveAspectRatio="" />
```

#### marker

marker不会在页面显示出来,一般定义在defs里面供复用使用,但是使用use元素复用是无法显示出来的,它只能显示在一个路径中的顶点位置,通过marker-start、marker-mid、marker-end样式属性引入

```xml
<path style="marker-start:url(#xxx)" />
```

marker中定义的元素是在marker这个独立坐标系中绘制的,所以要定义markerWidth、markerHeight属性,还可以指定marker相对于顶点位置的偏移,refX、refY,同时viewBox和preserveAspectRatio也是起作用的.

```xml
<marker id="" markerWidth="" markerHeight="" markerUnits="strokeWidth | userSpaceOnUse" refX="" refY="" orient="auto" viewBox="" preserveAspectRatio="" />
```

其中orient可以让标记自动旋转,匹配路径绘制方向,markerUnits值为strokeWidth时marker自身独立的坐标系统会随着笔画的粗细呈正相关.

### 样式指令

以上标签指令是svg中最基本和常用的几个形状标签,如果这些形状标签没有默认的有色样式,就会发现它们和HTML中的div一样,在页面上看不到任何东西,仅仅是有一些看不见的形状路径.要使用路径可以被视觉捕捉,就需要为路径添加颜色等样式.在svg中为路径添加样式有四种做法:

- 内联样式
- 内部样式表
- 外部样式表
- 样式属性

#### 内联样式

在style属性中书写样式

#### 内部样式表

在defs标签内部定义style标签

```xml
<defs>
  <style type="text/css">
    <![CDATA[
			circle{
				fill: yellow;
      }
		]]>
  </style>
</defs>
```

#### 外部样式表

把样式保存到外部的css文件中,如果svg作为一个单独的文件引用外部样式表时,则需要在头部定义一个stylesheet

```xml
<?xml-stylesheet href="xxx.css" type="text/css" ?>
```

而如果svg作为html页面的一部分时,则按html的规则正常引入

#### 表现属性

把样式作为属性名使用,这种优先级最低

```xml
<circle cx="100" cy="200" r="50" fill="yellow" stroke="red" />
```

与作画一样,对路径最基本的两种处理就是描边和填充.

#### 描边

#####  stroke-width

描边线条的宽度

##### stroke

描边线条的颜色

##### stroke-opacity

描边线条的透明度

##### stroke-dasharray

指定描边线条实线长度与空白间隔长度, 可以使用逗号或空格分隔

##### stroke-linecap

指定线条头尾形式,取值 butt | round | square

##### stroke-linejoin

指定两条线的连接处的样式 miter | round | bevel

##### stroke-miterlimit

![image-20201225143715958](https://github.com/jinjilynn/svg_tour/blob/master/image-20201225143715958.png)

当storke-linejoin为miter时有效

规定了斜接长度与线宽的比率

斜接长度就是两个锐角尖之间的距离

如果比率超过了stroke-miterlimit设置的值,stroke-linejoin就会变成bevel.

#### 填充

##### fill

填充颜色

##### fill-opacity

填充透明度

##### fill-rule

填充规则 evenodd | nonzero

规定了一个闭合的路径里哪些是属于内部区域可以被填充,哪些不是

nonzero与空间外围线条的绘制方向有关,顺时针+1,逆时针-1,总数为非0时则会被填充,总数为0时不被填充

evenodd与空间外围线条的个数有关,奇数则会被填充,偶数不被填充

### 路径

在HTML中,由于元素的绘制原理受制于盒模型,所以不能随心所欲的绘制各种形状,如果HTML想要绘制一些特殊的形状往往就需要一些奇技淫巧来完成.

相比于HTML,SVG就可以很容易的绘制出各种形状,因为SVG的绘制原理与HTML不同,它相当于把底层的绘制接口抛了出来,供使用者调用,前面提到的形状指令其实就是这种接口的快捷方式,这种绘制接口的使用方式就是路径指令标签\<path /\>.

```xml
<path d="" />
```

path指令可以绘制线段和曲线,曲线包括圆弧和贝塞尔曲线.其中所有的绘制信息都在d属性中定义.

d属性都以字符'M'或'm'开头,其后会紧跟使用空格或逗号分隔的x、y坐标,表示一个路径的起点.

d属性中可以包含多个'M'或'm',也就是多条不同形状的路径可以写在同一个d属性中.

#### 线段命令

从起点开始绘制线段有六种方法

- L

  大写字符'L'后面跟的是下一个点的绝对坐标值

  如果L命令后面的多个点都是L命令开头的点,则后面点的L可以省略

- l

  小写'l'后面跟的是上一个点的相对坐标,即相对于前一个点的坐标偏移量

  如果l命令后面的多个点都是l命令开头的点,则后面点的l可以胜率

- H

  大写'H'表示相对于前一个点的水平移动,后面跟一个x轴的坐标值

- h

  小写'h'也表示相对于前一个点的水平移动,后面跟一个相对于前一点的x轴偏移量

- V

  大写' V'表示相对于前一个点的垂直移动,后面跟一个y轴的坐标值

- v

  小写'v'也表示相对于前一个点的垂直移动,后面跟一个相对于前一点的y轴偏移量

#### 曲线命令

path路径绘制曲线有两种绘制方式: **椭圆弧** 与 **贝塞尔曲线**.

##### 椭圆弧

椭圆弧就是使用两个点确定某个椭圆上的一段弧线,圆弧命令以A或a开始,后面跟7个参数

- 椭圆x半径

- 椭圆y半径

- 椭圆x轴旋转角度

  - 如果旋转之后这两个点仍然在椭圆上,则会取到椭圆上不同部位的弧线

  如下图可以看到两个点是可以存在同一椭圆上的不同位置的,其中A是对椭圆进行x轴旋转后

  ![image-20201231141930326](https://github.com/jinjilynn/svg_tour/blob/master/image-20201231141930326.png)

- 椭圆上两点决定的大弧还是小弧 0 | 1 ( 0表示小弧;1表示大弧)

- 椭圆上的弧是以顺时针或逆时针绘制 0 | 1 (0表示逆时针;1表示顺时针)

  - 此时如果把整个椭圆都绘制出来会发现,顺时针和逆时针绘制是在对椭圆进行偏移后进行的,因为椭圆是对称的,椭圆上的任意两个点一定存在对称的两个点,而由于椭圆弧的起点和终点是固定的两组坐标,所以要改变绘制方向只有偏移整个椭圆,如下图同样两个端点按顺时针或逆时针绘制时,其实是在对椭圆进行偏移后绘制的.

    ![image-20201231141007837](https://github.com/jinjilynn/svg_tour/blob/master/image-20201231141007837.png)

- 终点x坐标

- 终点y坐标

椭圆弧两个端点的起点是A或a命令前面的那个点

以上七个参数加上起点就可以唯一确定一个圆弧上的弧线,但是如果指定的起点和终点不能同时出现在指定的椭圆上时,svg阅读器会缩放椭圆使两点在一个椭圆上,理论上不存在不同时在任意椭圆上的两个点

##### 贝塞尔曲线

贝塞尔曲线是曲线的优雅绘制.一条贝塞尔根据控制点个数的不同,可以分为二次贝塞尔曲线、三次贝塞尔曲线等多极贝塞尔曲线.

###### 二次贝塞尔曲线

二次贝塞尔曲线是有一个控制点的曲线,以Q或q命令开始,大写意味着绝对坐标,小写意味着相对坐标.

Q或q后面可以跟多组坐标点,其中每两个点为一组分别表示控制点和终点,这个终点可以作为下一组曲线的起点.

第一组的起点是Q或q前面的点

```xml
<path d="M30 100 Q80 30,100 100,135 60,200 80" />
```

在Q或q后面不止有一组点时,不是第一组的后面几组中的控制点可以省略并直接使用T或t代替,大写意味着绝对坐标,小写意味着相对坐标.

```xml
<path d="M30 100 Q80 30,100 100 T 200 80" />
```

svg阅读器会自动计算控制点并使两条曲线平滑过渡

###### 三次贝塞尔曲线

三次贝塞尔曲线是有两个控制点的曲线,以C或c命令开始,大写意味着绝对坐标,小写意味着相对坐标.

C或c后面可以跟多组坐标点,其中每三个点为一组分别表示第一个控制点、第二个控制点和终点,这个终点可以作为下一组曲线的起点.

第一组的起点时S或s前面的点

```xml
<path d="M30 90 C70 20,100,60,203 180,110 130,45 150,65 100" />
```

在C或c后面不止有一组点时,不是第一组的后面几组中的第一个控制点可以省略并直接使用S或s代替,大写意味着绝对坐标,小写意味着相对坐标.

```xml
<path d="M30 90 C70 20,100,60,203 180,S45 150,65 100" />
```

svg阅读器会自动计算第一个控制点并使两条曲线平滑过渡

**若对带宽有要求,以上所有坐标点中,数字和字母之间不需要空格,整数和负数之间也不需要空格**

### 文本

在计算机系统中文本的表达都涉及两个概念:字符集和字体.

- 字符集决定了文本的编码方式
- 字体决定了字符的视觉展现效果

在svg中文本使用Unicode字符集.并且要写在text标签命令中

#### text

```xml
<text x="x-value" y="y-value">xxxx</text>
```

text标签最基本的属性名是x和y,表示text标签中第一个字符的基线起点位置,文本的默认填充是黑色的,如果需要设置其他样式可使用style属性,其中的属性和css标准中是一样的:

- font-family
- font-size
- font-weight
- font-style
- text-decoration
- word-spacing
- letter-spacing

#### 对齐

text标签有两个属性可以设置水平或垂直的对齐方式:

- text-anchor
  - start | middle | end
- dominant-baseline
  - text-bottom | middle | text-top

#### tspan

以上的设置都是对text标签内的文本做统一的样式处理,如果需要对不同的文本应用不同的样式,除了可以把它们分别放在不同的text标签里外,还可以在text标签中使用tspan.

```xml
<tspan style="" dx="" dy="" x="" y="" rotate="">xxxxx</tspan>
```

可以为tspan单独指定style样式,或设置相对于元素本身的偏移dx、dy,或设置字符的旋转rotate,也可以设置片段的绝对位置型x、y,绝对位置可用于为文本进行换行.

其中dx、dy、x、y、rotate的值可以不是单独的一个值,也可以是用空格分隔的一组值,分别对应tspan内各个字符的偏移量.

**偏移对tspan标签结束后仍然会起作用**

dy在设置偏移量的时候可能需要一写计算会比较繁琐,这时可以使用baseline-shift属性快捷设置上标或下标:super | sub, 但这种属性只会在tspan内起作用,不会延续到tspan结束后.

#### 文本长度

textLength属性可以显示设置文本长度,同时配合lengthAdjust属性来调整字符大小或字符间距大小.但不推荐这么做,因为无论如何都是影响文字的外观美感.

#### writing-mode

writing-mode可以决定文本的书写方向默认为lr,如果需要从上往下书写则可以设置为tb.在svg中书写总想文本有两种方法,一种就是设置writing-mode,另一种是旋转坐标系90度.

文本变成纵向书写后如果想要每个字符仍然保持水平书写方向则可以设置glpy-orientation-vertical为0进行调整.

#### textPath

textPath是一个非常有用的指令,原来的文本只能直直的排列,而textPath可以让文本沿着任何path路径排列.

```xml
<text>
  <textPath xlink:href="" startOffset="">
    	xxxxxxxxxx
  </textPath>
</text>
```

xlink:href指定排列的路径, startOffset指定文本相对于路径开始位置的偏移,偏移量可以设置百分比或单位长度.

如果文本需要在路径中居中显示,可以在text上设置textancher="middle"并且在textPath上设置startOffset为50%









