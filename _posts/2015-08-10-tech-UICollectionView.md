---
layout: post
title: UICollectionView
category: 技术
tags: UICollectionView
keywords: UICollectionView
description:
---


### 解析`UICollectionViewController`

![UICollectionViewController](/assets/image/iOS-UICollectionViewController.png)

1. `UICollectionView`：类似于`UITableView`
2. `UICollectionViewCell`：类似于`UITableViewCell`
3. `Supplementary Views`：在cell外显示额外信息，通常用于`sections`的`headers`和`footers`
4. `Decoration View`：装饰视图(用作背景展示)
5. `UICollectionViewLayout`：UICollectionView的大脑和中枢，负责了将各个cell、Supplementary View和Decoration View进行组织，为它们设定各自的属性
6. `UICollectionViewFlowLayout`：Apple提供的继承`UICollectionViewLayout`的流式布局





### `UICollectionViewDataSource`

``` objc
- numberOfSectionsInCollectionView:

- collectionView:numberOfItemsInSection:// Required

- collectionView:cellForItemAtIndexPath:// Required

- collectionView:viewForSupplementaryElementOfKind:atIndexPath:
// UICollectionElementKindSectionHeader
// UICollectionElementKindSectionFooter
```



### `UICollectionViewDelegate`

``` objc
// Managing the Selected Cells
- collectionView:shouldSelectItemAtIndexPath://did
- collectionView:shouldDeselectItemAtIndexPath://did

// Managing Cell Highlighting
- collectionView:shouldHighlightItemAtIndexPath://did
- collectionView:didUnhighlightItemAtIndexPath:

// Tracking the Addition and Removal of Views
- collectionView:willDisplayCell:forItemAtIndexPath://didEnd
- collectionView:willDisplaySupplementaryView:forElementKind:atIndexPath://didEnd

// Providing a Transition Layout
- collectionView:transitionLayoutForOldLayout:newLayout:

// Managing Actions for Cells
- collectionView:shouldShowMenuForItemAtIndexPath:
- collectionView:canPerformAction:forItemAtIndexPath:withSender:
- collectionView:performAction:forItemAtIndexPath:withSender:
```



### `UICollectionViewLayout`

当`UICollectionView`在获取布局时将针对每一个`indexPath`的部件（包括cell，追加视图和装饰视图），向其上的`UICollectionViewLayout`实例询问该部件的布局信息，这个布局信息，以`UICollectionViewLayoutAttributes`的实例的方式给出。

``` objc
// UICollectionViewLayoutAttributes
@property (nonatomic) CGRect frame
@property (nonatomic) CGPoint center
@property (nonatomic) CGSize size
@property (nonatomic) CATransform3D transform3D
@property (nonatomic) CGFloat alpha
@property (nonatomic) NSInteger zIndex
@property (nonatomic, getter=isHidden) BOOL hidden
```



#### 自定义的`UICollectionViewLayout`

继承`UICollectionViewLayout`，然后重载下列方法：

``` objc
-(CGSize)collectionViewContentSize
// 返回collectionView的内容的尺寸

-(NSArray *)layoutAttributesForElementsInRect:(CGRect)rect
//返回rect中的所有的元素的布局属性,返回的是包含UICollectionViewLayoutAttributes的NSArray
//UICollectionViewLayoutAttributes可以是cell，追加视图或装饰视图的信息，通过不同的UICollectionViewLayoutAttributes初始化方法可以得到不同类型UICollectionViewLayoutAttributes：
//layoutAttributesForCellWithIndexPath:
//layoutAttributesForSupplementaryViewOfKind:withIndexPath:
//layoutAttributesForDecorationViewOfKind:withIndexPath:

-(UICollectionViewLayoutAttributes _)layoutAttributesForItemAtIndexPath:(NSIndexPath _)indexPath
// 返回对应于indexPath的位置的cell的布局属性

-(UICollectionViewLayoutAttributes _)layoutAttributesForSupplementaryViewOfKind:(NSString _)kind atIndexPath:(NSIndexPath *)indexPath
// 返回对应于indexPath的位置的追加视图的布局属性，如果没有追加视图可不重载

-(UICollectionViewLayoutAttributes * )layoutAttributesForDecorationViewOfKind:(NSString_)decorationViewKind atIndexPath:(NSIndexPath _)indexPath
// 返回对应于indexPath的位置的装饰视图的布局属性，如果没有装饰视图可不重载

-(BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds
// 当边界发生改变时，是否应该刷新布局。如果YES则在边界变化（一般是scroll到其他地方）时，将重新计算需要的布局信息。
```



在初始化一个`UICollectionViewLayout`实例后，会有一系列准备方法被自动调用，以保证layout实例的正确。

1. 首先，`-(void)prepareLayout`将被调用，默认下该方法什么没做，但是在自己的子类实现中，一般在该方法中设定一些必要的layout的结构和初始需要的参数等。
2. 之后，`-(CGSize) collectionViewContentSize`将被调用，以确定collection应该占据的尺寸。注意这里的尺寸不是指可视部分的尺寸，而**应该是所有内容所占的尺寸**。collectionView的本质是一个scrollView，因此需要这个尺寸来配置滚动行为。
3. 接下来`-(NSArray *)layoutAttributesForElementsInRect:(CGRect)rect`被调用。初始的layout的外观将由该方法返回的`UICollectionViewLayoutAttributes`来决定。
4. 另外，在**需要更新layout**时，需要给当前layout发送 **-invalidateLayout**，该消息会立即返回，并且**预约在下一个loop的时候刷新当前layout**，这一点和UIView的setNeedsLayout方法十分类似。在`-invalidateLayout`后的下一个collectionView的刷新loop中，又会从`prepareLayout`开始，依次再调用`-collectionViewContentSize`和`-layoutAttributesForElementsInRect`来生成更新后的布局。



- 案例：CircleLayout

``` objc
define ITEM_SIZE 70

@interface CircleLayout ()
@property(nonatomic, assign) CGPoint center;
@property(nonatomic, assign) CGFloat radius;
@property(nonatomic, assign) NSInteger cellCount;

// arrays to keep track of insert, delete index paths
@property (nonatomic, strong) NSMutableArray *deleteIndexPaths;
@property (nonatomic, strong) NSMutableArray *insertIndexPaths;
@end

@implementation CircleLayout
-(void)prepareLayout
{   //和init相似，必须call super的prepareLayout以保证初始化正确
    [super prepareLayout];

    CGSize size = self.collectionView.frame.size;
    _cellCount = [[self collectionView] numberOfItemsInSection:0];
    _center = CGPointMake(size.width / 2.0, size.height / 2.0);
    _radius = MIN(size.width, size.height) / 2.5;
}

//整个collectionView的内容大小就是collectionView的大小（没有滚动）
-(CGSize)collectionViewContentSize
{
    return [self collectionView].frame.size;
}

//通过所在的indexPath确定位置。
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)path
{
    UICollectionViewLayoutAttributes* attributes = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:path]; //生成空白的attributes对象，其中只记录了类型是cell以及对应的位置是indexPath
    //配置attributes到圆周上
    attributes.size = CGSizeMake(ITEM_SIZE, ITEM_SIZE);
    attributes.center = CGPointMake(_center.x + _radius * cosf(2 * path.item * M_PI / _cellCount), _center.y + _radius * sinf(2 * path.item * M_PI / _cellCount));
    return attributes;
}

//用来在一开始给出一套UICollectionViewLayoutAttributes
-(NSArray*)layoutAttributesForElementsInRect:(CGRect)rect
{
    NSMutableArray* attributes = [NSMutableArray array];
    for (NSInteger i=0 ; i < self.cellCount; i++) {
        //这里利用了-layoutAttributesForItemAtIndexPath:来获取attributes
        NSIndexPath* indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        [attributes addObject:[self layoutAttributesForItemAtIndexPath:indexPath]];
    }
    return attributes;
}
```



在对`collectionView`中的元素进行批量的插入，删除，移动等操作时，将触发`collectionView`所对应的`layout`的对应的动画。相应的动画由`layout`中的下列四个方法来定义：

`- initialLayoutAttributesForAppearingItemAtIndexPath:`

`- initialLayoutAttributesForAppearingDecorationElementOfKind:atIndexPath:`

`- finalLayoutAttributesForDisappearingItemAtIndexPath:`

`- finalLayoutAttributesForDisappearingDecorationElementOfKind:atIndexPath:`

在insert或者delete之前，`prepareForCollectionViewUpdates:`会被调用，可以使用这个方法来完成添加/删除的布局。

``` objc
- (void)prepareForCollectionViewUpdates:(NSArray *)updateItems
{
    // Keep track of insert and delete index paths
    [super prepareForCollectionViewUpdates:updateItems];

    self.deleteIndexPaths = [NSMutableArray array];
    self.insertIndexPaths = [NSMutableArray array];

    for (UICollectionViewUpdateItem *update in updateItems)
    {
        if (update.updateAction == UICollectionUpdateActionDelete)
        {
            [self.deleteIndexPaths addObject:update.indexPathBeforeUpdate];
        }
        else if (update.updateAction == UICollectionUpdateActionInsert)
        {
            [self.insertIndexPaths addObject:update.indexPathAfterUpdate];
        }
    }
}

- (void)finalizeCollectionViewUpdates
{
    [super finalizeCollectionViewUpdates];
    // release the insert and delete index paths
    self.deleteIndexPaths = nil;
    self.insertIndexPaths = nil;
}

// Also this gets called for all visible cells (not just the inserted ones) and
// even gets called when deleting cells!
- (UICollectionViewLayoutAttributes *)initialLayoutAttributesForAppearingItemAtIndexPath:(NSIndexPath *)itemIndexPath
{
    // Must call super
    UICollectionViewLayoutAttributes *attributes = [super initialLayoutAttributesForAppearingItemAtIndexPath:itemIndexPath];

    if ([self.insertIndexPaths containsObject:itemIndexPath])
    {
        // only change attributes on inserted cells
        if (!attributes)
            attributes = [self layoutAttributesForItemAtIndexPath:itemIndexPath];

        // Configure attributes ...
        attributes.alpha = 0.0;
        attributes.center = CGPointMake(_center.x, _center.y);
    }

    return attributes;
}

// Also this gets called for all visible cells (not just the deleted ones) and
// even gets called when inserting cells!
- (UICollectionViewLayoutAttributes *)finalLayoutAttributesForDisappearingItemAtIndexPath:(NSIndexPath *)itemIndexPath
{
    // So far, calling super hasn't been strictly necessary here, but leaving it in
    // for good measure
    UICollectionViewLayoutAttributes *attributes = [super finalLayoutAttributesForDisappearingItemAtIndexPath:itemIndexPath];

    if ([self.deleteIndexPaths containsObject:itemIndexPath])
    {
        // only change attributes on deleted cells
       if (!attributes)
            attributes = [self layoutAttributesForItemAtIndexPath:itemIndexPath];

        // Configure attributes ...
        attributes.alpha = 0.0;
        attributes.center = CGPointMake(_center.x, _center.y);
        attributes.transform3D = CATransform3DMakeScale(0.1, 0.1, 1.0);
    }

    return attributes;
}
```



### 参考

[http://onevcat.com/2012/08/advanced-collection-view/](http://onevcat.com/2012/08/advanced-collection-view/)



















