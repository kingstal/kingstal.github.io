### 自定义 View

- Core Graphics Concepts

> 1. **_Get a context_** to draw into (iOS will prepare one each time your `drawRect:` is called)
> 2. **_Create paths_** (out of lines, arcs, etc.)
> 3. **_Set colors, fonts, textures, linewidths, linecaps_**, etc.
> 4. **_Stroke or fill_** the above-created paths

`CGContextRef context = UIGraphicsGetCurrentContext();`


    {% highlight objective-c %}
    // 绘制 Gradient
    void drawLinearGradient(CGContextRef context, CGRect rect, CGColorRef startColor, CGColorRef endColor)
    {
        CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
        CGFloat locations[] = { 0.0, 1.0 };//起始和终止位置

        NSArray* colors = @[ (__bridge id)startColor, (__bridge id)endColor ];//开始颜色和结束颜色

        CGGradientRef gradient = CGGradientCreateWithColors(colorSpace, (__bridge CFArrayRef)colors, locations);// 渐变

        CGPoint startPoint = CGPointMake(CGRectGetMidX(rect), CGRectGetMinY(rect));
        CGPoint endPoint = CGPointMake(CGRectGetMidX(rect), CGRectGetMaxY(rect));// 渐变绘制的位置

        CGContextSaveGState(context);//保存上下文
        CGContextAddRect(context, rect);
        CGContextClip(context);//根据 path 裁剪
        CGContextDrawLinearGradient(context, gradient, startPoint, endPoint, 0);//绘制渐变
        CGContextRestoreGState(context);//恢复上下文

        CGGradientRelease(gradient);
        CGColorSpaceRelease(colorSpace);
    }
    {% endhighlight %}


    {% highlight objective-c %}
    // 绘制 Shadow
    CGContextSaveGState(context);
    CGContextSetShadowWithColor(context, CGSizeMake(0, 2), 3.0, shadowColor.CGColor);
    CGContextSetFillColorWithColor(context, self.lightColor.CGColor);
    CGContextFillRect(context, self.coloredBoxRect);
    CGContextRestoreGState(context);
    {% endhighlight %}


    {% highlight objective-c %}
    // 绘制 Gloss Effect：by applying a gradient alpha mask
    void drawGlossAndGradient(CGContextRef context, CGRect rect, CGColorRef startColor, CGColorRef endColor)
    {
        drawLinearGradient(context, rect, startColor, endColor);

        UIColor* glossColor1 = [UIColor colorWithRed:1.0 green:1.0 blue:1.0 alpha:0.35];
        UIColor* glossColor2 = [UIColor colorWithRed:1.0 green:1.0 blue:1.0 alpha:0.1];

        CGRect topHalf = CGRectMake(rect.origin.x, rect.origin.y, rect.size.width, rect.size.height / 2);

        drawLinearGradient(context, topHalf, glossColor1.CGColor, glossColor2.CGColor);
    }

    {% endhighlight %}


    {% highlight objective-c %}
    // 创建 arc
    CGMutablePathRef createArcPathFromBottomOfRect(CGRect rect, CGFloat arcHeight)
    {
        CGRect arcRect = CGRectMake(rect.origin.x, rect.origin.y + rect.size.height - arcHeight, rect.size.width, arcHeight);

        //    CGFloat arcRadius = (pow(arcRect.size.width / 2, 2) / arcRect.size.height + arcRect.size.height) / 2
        CGFloat arcRadius = (arcRect.size.height / 2) + (pow(arcRect.size.width, 2) / (8 * arcRect.size.height));
        CGPoint arcCenter
            = CGPointMake(arcRect.origin.x + arcRect.size.width / 2, arcRect.origin.y + arcRadius);

        //    CGFloat angle = acos(arcRect.size.width/2/arcRadius);
        CGFloat angle = acos(arcRect.size.width / (2 * arcRadius));
        CGFloat startAngle = radians(180) + angle;
        CGFloat endAngle = radians(360) - angle;

        CGMutablePathRef path = CGPathCreateMutable(); // 保存path用于复用
        CGPathAddArc(path, NULL, arcCenter.x, arcCenter.y, arcRadius, startAngle, endAngle, 0);
        CGPathAddLineToPoint(path, NULL, CGRectGetMaxX(rect), CGRectGetMinY(rect));
        CGPathAddLineToPoint(path, NULL, CGRectGetMinX(rect), CGRectGetMinY(rect));
        CGPathAddLineToPoint(path, NULL, CGRectGetMinX(rect), CGRectGetMaxY(rect));

        return path;
    }

    // 调用绘制 arc
    CGContextSaveGState(context);
    CGMutablePathRef arcPath = createArcPathFromBottomOfRect(arcRect, 4.0);
    CGContextAddPath(context, arcPath);
    CGContextClip(context);
    drawLinearGradient(context, paperRect, lightGrayColor.CGColor, darkGrayColor.CGColor);
    CGContextRestoreGState(context);

    // 绘制 arc 的另一种方法
    CGMutablePathRef createRoundedRectForRect(CGRect rect, CGFloat radius)
    {
        CGMutablePathRef path = CGPathCreateMutable();

        CGPathMoveToPoint(path, NULL, CGRectGetMidX(rect), CGRectGetMinY(rect));
        CGPathAddArcToPoint(path, NULL, CGRectGetMaxX(rect), CGRectGetMinY(rect), CGRectGetMaxX(rect), CGRectGetMaxY(rect), radius);
        CGPathAddArcToPoint(path, NULL, CGRectGetMaxX(rect), CGRectGetMaxY(rect), CGRectGetMinX(rect), CGRectGetMaxY(rect), radius);
        CGPathAddArcToPoint(path, NULL, CGRectGetMinX(rect), CGRectGetMaxY(rect), CGRectGetMinX(rect), CGRectGetMinY(rect), radius);
        CGPathAddArcToPoint(path, NULL, CGRectGetMinX(rect), CGRectGetMinY(rect), CGRectGetMaxX(rect), CGRectGetMinY(rect), radius);
        CGPathCloseSubpath(path);

        return path;
    }

    // Even-Odd Rule
    CGContextAddRect(context, paperRect);
    CGContextAddPath(context, arcPath);
    CGContextEOClip(context);// get outside based inside
    CGContextAddPath(context, arcPath);
    CGContextSetShadowWithColor(context, CGSizeMake(0, 2), 3.0, shadowColor.CGColor);
    CGContextFillPath(context);

    {% endhighlight %}

    {% highlight objective-c %}
    // CGContext Reference
    void CGContextSetLineWidth ( CGContextRef c, CGFloat width );
    void CGContextSetLineCap ( CGContextRef c, CGLineCap cap );
    void CGContextFillRect ( CGContextRef c, CGRect rect );
    void CGContextStrokeRect ( CGContextRef c, CGRect rect );
    void CGContextMoveToPoint ( CGContextRef c, CGFloat x, CGFloat y );
    void CGContextAddLineToPoint ( CGContextRef c, CGFloat x, CGFloat y );
    void CGContextStrokePath ( CGContextRef c );
    void CGContextSetShadowWithColor ( CGContextRef context, CGSize offset, CGFloat blur, CGColorRef color );
    {% endhighlight %}

- `UIBezierPath`

> Do all as `Core Graphics`, but capture it with an object.

> Then ask the object to stroke or fill what you’ve created.

    {% highlight objective-c %}
    // UIBezierPath draws into the current context, so don’t need to get it.
    UIBezierPath* path = [[UIBezierPath alloc] init];
    [path moveToPoint:CGPointMake(75, 10)];
    [path addLineToPoint:CGPointMake(160, 150)];
    [path addLineToPoint:CGPointMake(10, 150)];
    [path closePath];
    [[UIColor greenColor] setFill];
    [[UIColor redColor] setStroke];
    [path fill];
    [path stroke];

    UIBezierPath* roundedRect=[[UIBezierPath bezierPathWithRoundedRect:(CGRect) cornerRadius:(CGFloat):(CGRect)]];

    [roundedRect addClip]; // this would clip all drawing to be inside the roundedRect
    {% endhighlight %}



### Animation

- Animating views
> 1. Animating **specific properties**.
> 2. Animating **a group of changes** to a view “all at once.”
> 3. **Physics-based** animation.

Changes to certain UIView properties can be animated over time：`frame`、`transform` (translation, rotation and scale)、`alpha` (opacity)


    {% highlight objective-c %}
    +(void)animateWithDuration : (NSTimeInterval)duration
                        delay : (NSTimeInterval)delay
                      options : (UIViewAnimationOptions)options
                   animations : (void (^)(void))animations
                   completion : (void (^)(BOOL finished))completion
    {% endhighlight %}



    {% highlight objective-c %}
    + (void)transitionFromView:(UIView*)fromView
                    toView:(UIView*)toView
                  duration:(NSTimeInterval)duration
                   options:(UIViewAnimationOptions)options
                completion:(void (^)(BOOL finished))completion
    {% endhighlight %}


    {% highlight objective-c %}
    1.Create a UIDynamicAnimator
    2.Add UIDynamicBehaviors to it (gravity, collisions, etc.)
    3.Add UIDynamicItems (usually UIViews) to the UIDynamicBehaviors
    {% endhighlight %}

- Animation of View Controller transitions


- Core Animation
> Underlying powerful animation framework.
