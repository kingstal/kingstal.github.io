# 寄宿图
- contents：图片	`layer.contents = (__bridge id)image.CGImage;`
- contentsGravity：图片显示模式	  `layer.contentsGravity = kCAGravityResizeAspect;`
- contentsScale：图片缩放（适配 Retina）`layer.contentsScale = [UIScreen mainScreen].scale;`
- masksToBounds：是否显示超出边界的内容  `layer.masksToBounds = YES;`
- contentsRect：图片显示区域（矩形），使用单位坐标，默认(0,0,1,1)  `layer.contentsRect = CGRectMake(0, 0, 0.5, 0.5);`
- contentsCenter：是一个CGRect，它定义了一个固定的边框和一个在图层上可拉伸的区域，类似于`UIImage`的`resizableImageWithCapInsets:`  `layer.contentsCenter = CGRectMake(0.25, 0.25, 0.5, 0.5)`

# 图层几何学
- anchorPoint：图层的锚点  `layer.anchorPoint = CGPointMake(0.5f, 0.9f);`
- `CALayer`在不同坐标系剑的转换

```objective-c
- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer; 
- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer; 
- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;
```

- Z坐标轴：`zPosition`、`anchorPointZ`
- Hit Testing：`-containsPoint:`【需要将坐标转成每个图层坐标系下的坐标】、`-hitTest:`【`CALayer *layer = [self.layerView.layer hitTest:point];`】

# 视觉效果
- 圆角：`layer.cornerRadius = 20.0f;`
- 图层边框：`layer.borderWidth = 5.0f;`    `layer.borderColor = [UIColor greenColor].CGColor;`
- 阴影：`shadowOpacity`【在0.0（不可见）和1.0（完全不透明）之间的浮点数】、`shadowColor`、`shadowOffset`【控制阴影的方向和距离，是一个`CGSize`的值，宽度控制这阴影横向的位移，高度控制着纵向的位移】、`shadowRadius`【控制阴影的模糊度，当它的值是0的时候，阴影就和视图一样有一个非常确定的边界线。当值越来越大的时候，边界线看上去就会越来越模糊和自然】
- 阴影裁剪：如果想沿着内容裁切，需要用到两个图层：一个只画阴影的空的外图层，和一个用`masksToBounds`裁剪内容的内图层。
- shadowPath：`layer.shadowPath = squarePath;`
- 图层蒙板：`mask`本身就是个CALayer类型，定义了父图层的部分可见区域
- 拉伸过滤：`minificationFilter`【缩小】、`magnificationFilter`【放大】{kCAFilterLinear（默认）、kCAFilterNearest、kCAFilterTrilinear}
- 组透明：`shouldRasterize`实现组透明的效果，如果它被设置为YES，在应用透明度之前，图层及其子图层都会被整合成一个整体的图片。为了启用`shouldRasterize`属性，要设置图层的`rasterizationScale`属性。默认情况下，所有图层拉伸都是1.0， 所以如果使用了`shouldRasterize`属性，就要确保设置了`rasterizationScale`属性去匹配屏幕，以防止出现Retina屏幕像素化的问题。`layer.shouldRasterize = YES;    layer.rasterizationScale = [UIScreen mainScreen].scale;`

# 变换
## 仿射变换(`CGAffineTransform`)【平移、旋转、缩放、斜切】
`UIView`可以通过设置`transform`属性做变换，但实际上它只是封装了内部图层的变换。`CALayer`同样也有一个`transform`属性，但它的类型是`CATransform3D`，而不是`CGAffineTransform`，`CALayer`对应于`UIView`的`transform`属性叫做`affineTransform`。
![仿射变换.png](/assets/image/iOS-Core-Animation-Advanced-Techniques-仿射变换.png)

```objective-c
CGAffineTransformMakeRotation(CGFloat angle) 
CGAffineTransformMakeScale(CGFloat sx, CGFloat sy)
CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty)
```

- 混合变换：在一个变换的基础上做更深层次的变换

```Objective-C
CGAffineTransformRotate(CGAffineTransform t, CGFloat angle)     
CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy)      
CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)

CGAffineTransformIdentity//单位矩阵
CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);//在两个变换的基础上创建一个新的变换
```

- 斜切变换：不常用

## 3D 变换
![3D 变换](/assets/image/iOS-Core-Animation-Advanced-Techniques-3D变换.png)

```objective-c
CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)
CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz) 
CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)
```

- 透视投影：`CATransform3D`的透视效果通过矩阵中一个很简单的元素来控制：`m34`。`m34`用于按比例缩放X和Y的值来计算到底要离视角多远。`m34`的默认值是0，我们可以通过设置`m34`为`-1.0 / d`来应用透视效果，`d`代表了想象中视角相机和屏幕之间的距离，以像素为单位，那应该如何计算这个距离呢？实际上并不需要，大概估算一个就好了。因为视角相机实际上并不存在，所以可以根据屏幕上的显示效果自由决定它的放置的位置。通常500-1000就已经很好了，但对于特定的图层有时候更小或者更大的值会看起来更舒服，减少距离的值会增强透视效果，所以一个非常微小的值会让它看起来更加失真，然而一个非常大的值会让它基本失去透视效果。
- 灭点：当在透视角度绘图的时候，远离相机视角的物体将会变小变远，当远离到一个极限距离，它们可能就缩成了一个点，于是所有的物体最后都汇聚消失在同一个点。做3D变换的时候要时刻记住这一点，当视图通过调整m34来让它更加有3D效果，应该首先把它放置于屏幕中央，然后通过平移来把它移动到指定位置（而不是直接改变它的position），这样所有的3D图层都共享一个灭点。
- `sublayerTransform`属性：是`CATransform3D`类型，它影响到所有的子图层。这意味着可以一次性对包含这些图层的容器做变换，于是所有的子图层都自动继承了这个变换方法。可以通过设置父视图的透视变换，可以保证子视图有相同的透视和灭点。
- 背面：图层是双面绘制的，反面显示的是正面的一个镜像图片。`doubleSided`的属性来控制图层的背面是否要被绘制


