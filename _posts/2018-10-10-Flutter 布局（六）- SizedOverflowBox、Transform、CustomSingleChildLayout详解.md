---
layout:     post
title:      Flutter 布局（六）- SizedOverflowBox、Transform、CustomSingleChildLayout详解
subtitle:   主要介绍Flutter布局中的SizedOverflowBox、Transform、CustomSingleChildLayout三种控件
date:       2018-10-15
author:     ZXL
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - Flutter
---
# Flutter 布局（六）- SizedOverflowBox、Transform、CustomSingleChildLayout详解

> 本文主要介绍Flutter布局中的SizedOverflowBox、Transform、CustomSingleChildLayout三种控件，详细介绍了其布局行为以及使用场景，并对源码进行了分析。

## 1. SizedOverflowBox

> A widget that is a specific size but passes its original constraints through to its child, which will probably overflow.

### 1.1 简介

光看名称，就可以猜出，SizedOverflowBox是SizedBox与OverflowBox的结合体。

### 1.2 布局行为

SizedOverflowBox主要的布局行为有两点：

* 尺寸部分。通过将自身的固定尺寸，传递给child，来达到控制child尺寸的目的；
* 超出部分。可以突破父节点尺寸的限制，超出部分也可以被渲染显示，与OverflowBox类似。

### 1.3 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > SizedOverflowBox
```

### 1.4 示例代码

```
Container(
  color: Colors.green,
  alignment: Alignment.topRight,
  width: 200.0,
  height: 200.0,
  padding: EdgeInsets.all(5.0),
  child: SizedOverflowBox(
    size: Size(100.0, 200.0),
    child: Container(color: Colors.red, width: 200.0, height: 100.0,),
  ),
);
```

代码运行的时候报出了下面的异常，很神奇。但是同学们可以自己看看代码运行的效果，可以超出，但是不太好用。

```
 ══╡ EXCEPTION CAUGHT BY RENDERING LIBRARY ╞═════════════════════════════════════════════════════════
```

### 1.5 源码解析

```
const SizedOverflowBox({
  Key key,
  @required this.size,
  this.alignment = Alignment.center,
  Widget child,
})
```

#### 1.5.1 属性解析

**size**：固定的尺寸。

**alignment**：对齐方式。


#### 1.5.2 源码

直接上布局相关的代码：

```
size = constraints.constrain(_requestedSize);
if (child != null) {
  child.layout(constraints);
  alignChild();
}
```

如果child存在的话，就将child设为对应的尺寸，然后按照对齐方式进行对齐。但是在实际写sample的时候，感觉跟我预想中的表现不太一致，而且经常报出异常。不知道是我理解错了，还是样例写错了，如果有了解的同学，麻烦告知，在此感谢。

### 1.6 使用场景

代码的表现跟我预想中的不太一致，更推荐使用OverflowBox，场景也跟其比较一致。

## 2. Transform

> A widget that applies a transformation before painting its child.

### 2.1 简介

Transform在介绍Container的时候有提到过，就是做矩阵变换的。Container中矩阵变换就是使用的Transform。

### 2.2 布局行为

有过其他平台经验的，对Transform应该不会陌生。可以对child做平移、旋转、缩放等操作。

### 2.3 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Transform
```

### 2.4 示例代码

```
Center(
  child: Transform(
    transform: Matrix4.rotationZ(0.3),
    child: Container(
      color: Colors.blue,
      width: 100.0,
      height: 100.0,
    ),
  ),
)
```

示例中将Container绕z轴旋转了，代码很简单。Matrix4也提供了很多便捷的构造函数供大家使用，因此写起来，并不会有太大的难度。

### 2.5 源码解析

```
const Transform({
  Key key,
  @required this.transform,
  this.origin,
  this.alignment,
  this.transformHitTests = true,
  Widget child,
})
```

上面是其默认的构造函数，Transform也提供下面三种构造函数：

```
Transform.rotate
Transform.translate
Transform.scale
```

#### 2.5.1 属性解析

**transform**：一个4x4的矩阵。不难发现，其他平台的变换矩阵也都是四阶的。一些复合操作，仅靠三维是不够的，必须采用额外的一维来补充，感兴趣的同学可以自行搜索了解。

**origin**：旋转点，相对于左上角顶点的偏移。默认旋转点事左上角顶点。

**alignment**：对齐方式。

**transformHitTests**：点击区域是否也做相应的改变。

#### 2.5.2 源码

我们来看看它的绘制代码：

```
if (child != null) {
  final Matrix4 transform = _effectiveTransform;
  final Offset childOffset = MatrixUtils.getAsTranslation(transform);
  if (childOffset == null)
    context.pushTransform(needsCompositing, offset, transform, super.paint);
  else
    super.paint(context, offset + childOffset);
}
```

整个绘制代码不复杂，如果child有偏移的话，则将两个偏移相加，进行绘制。如果child没有偏移的话，则按照设置的offset、transform进行绘制。


### 2.6 使用场景

这个控件算是较常见的控件，很多平移、旋转、缩放都可以使用的到。如果只是单纯的进行变换的话，用Transform比用Container效率会更高。

## 3. CustomSingleChildLayout

> A widget that defers the layout of its single child to a delegate.

### 3.1 简介

一个通过外部传入的布局行为，来进行布局的控件，不同于其他固定布局的控件，我们自定义一些单节点布局控件的时候，可以考虑使用它。

### 3.2 布局行为

CustomSingleChildLayout提供了一个控制child布局的delegate，这个delegate可以控制这些因素：

* 可以控制child的布局constraints；
* 可以控制child的位置；
* 在parent的尺寸不依赖于child的情况下，可以决定parent的尺寸。

### 3.3 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > CustomSingleChildLayout
```

### 3.4 示例代码

```
class FixedSizeLayoutDelegate extends SingleChildLayoutDelegate {
  FixedSizeLayoutDelegate(this.size);

  final Size size;

  @override
  Size getSize(BoxConstraints constraints) => size;

  @override
  BoxConstraints getConstraintsForChild(BoxConstraints constraints) {
    return new BoxConstraints.tight(size);
  }

  @override
  bool shouldRelayout(FixedSizeLayoutDelegate oldDelegate) {
    return size != oldDelegate.size;
  }
}

Container(
  color: Colors.blue,
  padding: const EdgeInsets.all(5.0),
  child: CustomSingleChildLayout(
    delegate: FixedSizeLayoutDelegate(Size(200.0, 200.0)),
    child: Container(
      color: Colors.red,
      width: 100.0,
      height: 300.0,
    ),
  ),
)
```

由于SingleChildLayoutDelegate是一个抽象类，我们需要单独写一个继承自它的delegate，来进行相关的布局操作。上面例子中是一个固定尺寸的布局。

### 3.5 源码解析

构造函数如下：

```
const CustomSingleChildLayout({
  Key key,
  @required this.delegate,
  Widget child
})
```

#### 3.5.1 属性解析

**alignment**：一个抽象类，我们需要自行实现布局的类。

#### 3.5.2 源码

我们直接来看其布局函数：

```
size = _getSize(constraints);
if (child != null) {
  final BoxConstraints childConstraints = delegate.getConstraintsForChild(constraints);
  assert(childConstraints.debugAssertIsValid(isAppliedConstraint: true));
  child.layout(childConstraints, parentUsesSize: !childConstraints.isTight);
  final BoxParentData childParentData = child.parentData;
  childParentData.offset = delegate.getPositionForChild(size, childConstraints.isTight ? childConstraints.smallest : child.size);
}
```

其child的constraints是通过delegate传入的，而这个delegate则是我们通过外部继承自SingleChildLayoutDelegate实现的。

我们接下来看一下SingleChildLayoutDelegate提供了哪些接口：

```
Size getSize(BoxConstraints constraints) => constraints.biggest;
BoxConstraints getConstraintsForChild(BoxConstraints constraints) => constraints;
Offset getPositionForChild(Size size, Size childSize) => Offset.zero;
bool shouldRelayout(covariant SingleChildLayoutDelegate oldDelegate);
```

其中前三个都是布局相关的，包括尺寸、constraints、位置。最后一个函数是判断是否需要重新布局的。我们在编写delegate的时候，可以选择需要进行修改的属性进行重写，并给予相应的布局属性即可。

### 3.6 使用场景

这种控件用起来可能会繁琐一些，但是我们可以通过这个控件去封装一些基础的控件，供他人使用。

## 4. 阶段性小结

到目前为止，Flutter中单子节点布局控件，大致上都简单的梳理了一遍。如果一直在关注这个系列文章的同学，应该可以发现我经常吐槽其控件设计。

Flutter中总共有18个单子节点布局控件，18个啊，还没有算多子节点布局控件，也没有算可能会新增的。这样的学习成本非常高。虽然常见的就那么几种，平时一直用那些也都没有问题，但是布局的时候总是会遇到那么几种解决不了的问题。而且梳理的时候，会发现，几种控件都能解决的问题，并不是说把效果实现出了就完事了，这中间肯定会涉及到哪种控件效率更高。如果要对Flutter项目做较深入的性能优化，这些控件肯定都得掌握。

Flutter的这种设计，把一些原本应该由它们承担的成本，转移到了开发者身上。很多控件在日常使用中几乎都用不上，并没有考虑太多实际的使用场景。当然了，也还是得安慰自己，这毕竟只是初期，乱点就乱点，日子肯定会越来越好的。

后面还会将多子节点控件全部梳理一遍，然后来个大总结，对什么场景该使用哪种控件，如何进行控件级别的优化，做一个总结。

## 5. 后话

笔者建了一个Flutter学习相关的项目，[Github地址](https://github.com/yang7229693/flutter-study)，里面包含了笔者写的关于Flutter学习相关的一些文章，会定期更新，也会上传一些学习Demo，欢迎大家关注。

##  6. 参考

1. [SizedOverflowBox class](https://docs.flutter.io/flutter/widgets/SizedOverflowBox-class.html)
2. [Transform class](https://docs.flutter.io/flutter/widgets/Transform-class.html)
3. [CustomSingleChildLayout class](https://docs.flutter.io/flutter/widgets/CustomSingleChildLayout-class.html)


