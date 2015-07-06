---
layout: post
title: Facebook Pop Tutorial
category: 技术
tags: Facebook Pop
keywords: Facebook Pop
description: Facebook Pop Tutorial
---


转载[https://github.com/maxmyers/FacebookPop/blob/master/README.md](https://github.com/maxmyers/FacebookPop/blob/master/README.md)


# 5 Steps For Using Facebook Pop

```objc
  // 1. Pick a Kind Of Animation 
  //  POPBasicAnimation  POPSpringAnimation POPDecayAnimation
  POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];

  // 2. Decide weather you will animate a view property or layer property, Lets pick a View Property and pick kPOPViewFrame
  // View Properties - kPOPViewAlpha kPOPViewBackgroundColor kPOPViewBounds kPOPViewCenter kPOPViewFrame kPOPViewScaleXY kPOPViewSize
  // Layer Properties - kPOPLayerBackgroundColor kPOPLayerBounds kPOPLayerScaleXY kPOPLayerSize kPOPLayerOpacity kPOPLayerPosition kPOPLayerPositionX kPOPLayerPositionY kPOPLayerRotation kPOPLayerBackgroundColor
  basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewFrame];

  // 3. Figure Out which of 3 ways to set toValue
  //  anim.toValue = @(1.0);
  //  anim.toValue =  [NSValue valueWithCGRect:CGRectMake(0, 0, 400, 400)];
  //  anim.toValue =  [NSValue valueWithCGSize:CGSizeMake(40, 40)];
  basicAnimation.toValue=[NSValue valueWithCGRect:CGRectMake(0, 0, 90, 190)];

  // 4. Create Name For Animation & Set Delegate
   basicAnimation.name=@"AnyAnimationNameYouWant";
   basicAnimation.delegate=self

  // 5. Add animation to View or Layer, we picked View so self.tableView and not layer which would have been self.tableView.layer
  [self.tableView pop_addAnimation:basicAnimation forKey:@"WhatEverNameYouWant"];
```


## Step 1 Pick Kind of Animation

### POPBasicAnimation
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.timingFunction=[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
// kCAMediaTimingFunctionLinear  kCAMediaTimingFunctionEaseIn  kCAMediaTimingFunctionEaseOut  kCAMediaTimingFunctionEaseInEaseOut
```

### POPSpringAnimation
```objc
POPSpringAnimation *springAnimation = [POPSpringAnimation animation];
springAnimation.velocity=@(1000);       // change of value units per second
springAnimation.springBounciness=14;    // value between 0-20 default at 4
springAnimation.springSpeed=3;     // value between 0-20 default at 4
```

### POPDecayAnimation
```objc
POPDecayAnimation *decayAnimation=[POPDecayAnimation animation];
decayAnimation.velocity=@(233); //change of value units per second
decayAnimation.deceleration=.833; //range of 0 to 1
```

### Example
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
```

## Step 2 Decide if you will animate a view property or layer property

### View Properties

- Alpha - kPOPViewAlpha
- Color - kPOPViewBackgroundColor
- Size - kPOPViewBounds
- Center - kPOPViewCenter
- Location & Size - kPOPViewFrame
- Size - kPOPViewScaleXY
- Size(Scale) - kPOPViewSize


### Layer Properties

- Color - kPOPLayerBackgroundColor
- Size - kPOPLayerBounds
- Size - kPOPLayerScaleXY
- Size - kPOPLayerSize
- Opacity - kPOPLayerOpacity
- Position - kPOPLayerPosition
- X Position - kPOPLayerPositionX
- Y Position - kPOPLayerPositionY
- Rotation - kPOPLayerRotation
- Color - kPOPLayerBackgroundColor

### Example
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
```

**Note**: Works on any Layer property or any object that inherits from UIView such as `UIToolbar` 、 `UIPickerView` 、 `UIDatePicker` 、 `UIScrollView` 、  `UITextView` 、 `UIImageView` 、 `UITableViewCell` 、 `UIStepper` 、 `UIProgressView` 、 `UIActivityIndicatorView` 、 `UISwitch` 、 `UISlider` 、 `UITextField` 、 `UISegmentedControl` 、 `UIButton` 、 `UIView` 、 `UITableView`


## Step 3 Find your property below then add and set .toValue

### View Properties

##### Alpha - kPOPViewAlpha
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewAlpha];
basicAnimation.toValue= @(0); // scale from 0 to 1
```

##### Color - kPOPViewBackgroundColor
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPViewBackgroundColor];
basicAnimation.toValue= [UIColor redColor];
```

##### Size - kPOPViewBounds
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewBounds];
basicAnimation.toValue=[NSValue valueWithCGRect:CGRectMake(0, 0, 90, 190)]; //first 2 values dont matter
```

##### Center - kPOPViewCenter
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewCenter];
basicAnimation.toValue=[NSValue valueWithCGPoint:CGPointMake(200, 200)];
```

##### Location & Size - kPOPViewFrame
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewFrame];
basicAnimation.toValue=[NSValue valueWithCGRect:CGRectMake(140, 140, 140, 140)];
```

##### Size - kPOPViewScaleXY
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewScaleXY];
basicAnimation.toValue=[NSValue valueWithCGSize:CGSizeMake(3, 2)];
```

##### Size(Scale) - kPOPViewSize
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewSize];
basicAnimation.toValue=[NSValue valueWithCGSize:CGSizeMake(30, 200)];
```

### Layer Properties


##### Color - kPOPLayerBackgroundColor
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerBackgroundColor];
basicAnimation.toValue= [UIColor redColor];
```

##### Size - kPOPLayerBounds
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerBounds];
basicAnimation.toValue= [NSValue valueWithCGRect:CGRectMake(0, 0, 90, 90)]; //first 2 values dont matter
```

##### Size - kPOPLayerScaleXY
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerScaleXY];
basicAnimation.toValue= [NSValue valueWithCGSize:CGSizeMake(2, 1)];//increases width and height scales
```

##### Size - kPOPLayerSize
```objc
POPBasicAnimation *basicAnimation = [POPBasicAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPLayerSize];
basicAnimation.toValue= [NSValue valueWithCGSize:CGSizeMake(200, 200)];
```

##### Opacity - kPOPLayerOpacity
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerOpacity];
basicAnimation.toValue = @(0);
```

##### Position - kPOPLayerPosition
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerPosition];
basicAnimation.toValue= [NSValue valueWithCGRect:CGRectMake(130, 130, 0, 0)];//last 2 values dont matter
```

##### X Position - kPOPLayerPositionX
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerPositionX];
basicAnimation.toValue= @(240);
```

##### Y Position - kPOPLayerPositionY
```objc
POPSpringAnimation *anim = [POPSpringAnimation animation];
anim.property=[POPAnimatableProperty propertyWithName:kPOPLayerPositionY];
anim.toValue = @(320);
```

##### Rotation - kPOPLayerRotation
```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerRotation];
basicAnimation.toValue= @(M_PI/4); //2 M_PI is an entire rotation
```

**Note**: Property Changes work for all 3 animation types - POPBasicAnimation POPSpringAnimation POPDecayAnimation

### Example

```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName: kPOPLayerRotation];
basicAnimation.toValue= @(M_PI/4); //2 M_PI is an entire rotation
```

## Step 4 Create Name & Delegate For Animation
```objc
basicAnimation.name=@"WhatEverAnimationNameYouWant";
basicAnimation.delegate=self;
```

Declare Delegate Protocol `<POPAnimatorDelegate>`

### Delegate Methods

`<POPAnimatorDelegate>`

```objc
- (void)pop_animationDidStart:(POPAnimation *)anim;
```

```objc
- (void)pop_animationDidStop:(POPAnimation *)anim finished:(BOOL)finished;
```

```objc
- (void)pop_animationDidReachToValue:(POPAnimation *)anim;
```


### Example

```objc
POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewFrame];
basicAnimation.toValue=[NSValue valueWithCGRect:CGRectMake(0, 0, 90, 190)];
basicAnimation.name=@"WhatEverAnimationNameYouWant";
basicAnimation.delegate=self;
```

## Step 5 Add animation to View

```objc
 [self.tableView pop_addAnimation:basicAnimation forKey:@"WhatEverNameYouWant"];
```

### Example
```objc
  POPSpringAnimation *basicAnimation = [POPSpringAnimation animation];
  basicAnimation.property = [POPAnimatableProperty propertyWithName:kPOPViewFrame];
  basicAnimation.toValue=[NSValue valueWithCGRect:CGRectMake(0, 0, 90, 190)];
  basicAnimation.name=@"SomeAnimationNameYouChoose";
  basicAnimation.delegate=self;
  [self.tableView pop_addAnimation:basicAnimation forKey:@"WhatEverNameYouWant"];
```

