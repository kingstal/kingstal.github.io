### Applying Tint Colors

``` swift
let sharedApplication = UIApplication.sharedApplication()

sharedApplication.delegate?.window??.tintColor = theme.mainColor
```

### Customizing the Navigation Bar

``` swift
UINavigationBar.appearance().barStyle = theme.barStyle

UINavigationBar.appearance().setBackgroundImage(theme.navigationBackgroundImage, forBarMetrics: .Default)
```

> UIKit has an informal protocol called `UIAppearance` that most of its controls conform to. When you call `appearance()` **on UIKit classes**— not instances —it **returns a UIAppearance proxy**. When you change the properties of this proxy, all the instances of that class automatically get the same value.

### Customizing the Navigation Bar Back Indicator

> As it turns out, images in iOS have three rendering modes:
> 
> **Original**: Always use the image “as is” with its original colors.
> 
> **Template**: Ignore the colors, and just use the image as a stencil. In this mode, iOS uses only the shape of the image, and colors the image itself before rendering it on screen. So when a control has a tint color, iOS takes the shape from the image you provide and uses the tint color to color it.
> 
> **Automatic**: Depending on the context in which you use the image, the system decides whether it should draw the image as “original” or “template”. For items such as back indicators, navigation control bar button items and tab bar images, iOS ignores the image colors by default unless you change the rendering mode.

``` swift
UINavigationBar.appearance().backIndicatorImage = UIImage(named: "backArrow")
        UINavigationBar.appearance().backIndicatorTransitionMaskImage = UIImage(named: "backArrowMask")
```

### Customizing the Tab Bar

``` swift
UITabBar.appearance().barStyle = theme.barStyle
        UITabBar.appearance().backgroundImage = theme.tabBarBackgroundImage
        
        let tabIndicator = UIImage(named: "tabBarSelectionIndicator")?.imageWithRenderingMode(.AlwaysTemplate)
        let tabResizableIndicator = tabIndicator?.resizableImageWithCapInsets(UIEdgeInsets(top: 0, left: 2.0, bottom: 0, right: 2.0))
        UITabBar.appearance().selectionIndicatorImage = tabResizableIndicator
```

### Customizing a Segmented Control

``` swift
let controlBackground = UIImage(named: "controlBackground")?
            .imageWithRenderingMode(.AlwaysTemplate)
            .resizableImageWithCapInsets(UIEdgeInsets(top: 3, left: 3, bottom: 3, right: 3))
        let controlSelectedBackground = UIImage(named: "controlSelectedBackground")?
            .imageWithRenderingMode(.AlwaysTemplate)
            .resizableImageWithCapInsets(UIEdgeInsets(top: 3, left: 3, bottom: 3, right: 3))
        
        UISegmentedControl.appearance().setBackgroundImage(controlBackground, forState: .Normal,
            barMetrics: .Default)
        UISegmentedControl.appearance().setBackgroundImage(controlSelectedBackground, forState: .Selected, barMetrics: .Default)
```

### Customizing Steppers, Sliders, and Switches

``` swift
UIStepper.appearance().setBackgroundImage(controlBackground, forState: .Normal)
UIStepper.appearance().setBackgroundImage(controlBackground, forState: .Disabled)
UIStepper.appearance().setBackgroundImage(controlBackground, forState: .Highlighted)
UIStepper.appearance().setDecrementImage(UIImage(named: "fewerPaws"), forState: .Normal)//DecrementImage
UIStepper.appearance().setIncrementImage(UIImage(named: "morePaws"), forState: .Normal)//IncrementImage

UISlider.appearance().setThumbImage(UIImage(named: "sliderThumb"), forState: .Normal)//thumb
UISlider.appearance().setMaximumTrackImage(UIImage(named: "maximumTrack")?
    .resizableImageWithCapInsets(UIEdgeInsets(top: 0, left: 0.0, bottom: 0, right: 6.0)), 
      forState: .Normal)//minimum track
UISlider.appearance().setMinimumTrackImage(UIImage(named: "minimumTrack")?
    .imageWithRenderingMode(.AlwaysTemplate)
      .resizableImageWithCapInsets(UIEdgeInsets(top: 0, left: 6.0, bottom: 0, right: 0)), 
        forState: .Normal)//maximum track
 
UISwitch.appearance().onTintColor = theme.mainColor.colorWithAlphaComponent(0.3)//onTintColor
UISwitch.appearance().thumbTintColor = theme.mainColor//thumbTintColor
```









