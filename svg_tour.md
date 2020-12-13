# About SVG

可缩放矢量图,一种XML应用

# 图形系统

- ## 栅格图形

  图形是由单个像素拼凑而成,存在形式为jpg、png等二进制文件

- ## 矢量图形

  图形被描述为一系列的指令,以XML文件形式存在

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

所有的变换类型都写在元素的transform属性中

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