---
title: iOS 瀑布流布局实现详解
date: 2019-02-18 18:16:00
tags: 代码库
---

新建文件继承自UICollectionViewLayout
.h内容如下：
```
@class WaterFlowLayout;
@protocol WaterFlowLayoutDelegate <NSObject>

//使用delegate取得每一个Cell的高度
- (CGFloat)waterFlow:(WaterFlowLayout *)layout heightForCellAtIndexPath:(NSIndexPath *)indexPath;

@end

@interface WaterFlowLayout : UICollectionViewLayout

//声明协议
@property (nonatomic, weak) id <WaterFlowLayoutDelegate> delegate;
//确定列数
@property (nonatomic, assign) NSInteger colum;
//确定内边距
@property (nonatomic, assign) UIEdgeInsets insetSpace;
//确定每个cell之间的距离
@property (nonatomic, assign) NSInteger distance;

@end
```
.m实现内容如下：
```
@interface WaterFlowLayout ()
//存储列高的数组
@property (nonatomic, strong) NSMutableArray *columHeightArr;
//存储所有cell的尺寸信息
@property (nonatomic, strong) NSMutableArray *cellFrameArr;

@end

@implementation WaterFlowLayout

//colum的set方法
- (void)setColum:(NSInteger)colum{
    if (_colum != colum) {
        _colum = colum;
        //将之前的布局信息失效，重新布局
        [self invalidateLayout];
    }
}

//distance的set方法
- (void)setDistance:(NSInteger)distance{
    if (_distance != distance) {
        _distance = distance;
        [self invalidateLayout];
    }
}

//insetSpace的set方法
- (void)setInsetSpace:(UIEdgeInsets)insetSpace{
    if (!UIEdgeInsetsEqualToEdgeInsets(_insetSpace, insetSpace)) {
        _insetSpace = insetSpace;
        [self invalidateLayout];
    }
}

//自定义layout需要重写下面的几个方法
//准备布局，将item的位置信息计算出来
- (void)prepareLayout{
    //将位置信息和高度信息的数组实例化
    [self initDataArray];
    //初始化每一列的初始高度
    [self initColumHeightArray];
    //初始化计算出全部cell的高度，并且存入数组
    [self initAllCellHeight];
}

//将位置信息和高度信息的数组实例化
- (void)initDataArray{
    //记录当前每一列的高度，所以我们只需要列数的空间就够了。
    _columHeightArr = [NSMutableArray arrayWithCapacity:_colum];
    //记录所有cell的尺寸信息
    _cellFrameArr = [NSMutableArray arrayWithCapacity:0];
}

//初始化每一列的初始高度
- (void)initColumHeightArray{
    for (int i = 0; i < _colum; i++) {
        [_columHeightArr addObject:@(_insetSpace.top)];
    }
}

//初始化计算出全部cell的高度，并且存入数组
- (void)initAllCellHeight{
    //拿出第一组的全部cell的数量
    NSInteger allCellNumber = [self.collectionView numberOfItemsInSection:0];
    //取得整个collectionView的宽度
    CGFloat totalWidth = self.collectionView.frame.size.width;
    //取得一行中Cell的总宽度
    CGFloat itemAllWidth = totalWidth - _insetSpace.left - _insetSpace.right - _distance * (_colum - 1);
    //取得每一个cell的宽度
    CGFloat width = itemAllWidth/_colum;
    //循环计算每一个cell的高度并且将位置信息添加到数组中
    for (int i = 0; i < allCellNumber; i++) {
        //拿到当前的列的信息
        NSInteger currentColum = [self getShortColum];
        //x偏移就是用当前的列去乘以宽度和间距，并且加上内边距
        CGFloat xOffset = _insetSpace.left + currentColum * (width + _distance);
        //制造索引路径
        NSIndexPath *indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        //取得y偏移
        CGFloat yOffset = [[_columHeightArr objectAtIndex:currentColum] floatValue] + _distance;
        //取得高度，由实现协议者提供
        CGFloat height = 0.f;
        if (_delegate && [_delegate respondsToSelector:@selector(waterFlow:heightForCellAtIndexPath:)]) {
            height = [_delegate waterFlow:self heightForCellAtIndexPath:indexPath];
        }
        //整理cell的尺寸信息
        CGRect frame = CGRectMake(xOffset, yOffset, width, height);
        //attributes是用来存储当前indexPath的cell的位置信息的
        UICollectionViewLayoutAttributes *attributes = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
        attributes.frame = frame;
        //将位置信息添加到cell尺寸数组中
        [_cellFrameArr addObject:attributes];
        //改变当前列的高度
        _columHeightArr[currentColum] = @(frame.size.height + frame.origin.y);
    }
}

//取得当前cell的尺寸
-(UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
    return [_cellFrameArr objectAtIndex:indexPath.item];
}

//根据rect去找出需要布局的cell的位置信息
- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect{
    //用来存储可以展示的cell的位置信息
    NSMutableArray *temp = [NSMutableArray arrayWithCapacity:0];
    for (UICollectionViewLayoutAttributes *attributes in _cellFrameArr) {
        //如果取出的位置信息，在rect的范围内，就将这个位置信息，装入数组中。
        if (CGRectIntersectsRect(attributes.frame, rect)) {
            [temp addObject:attributes];
        }
    }
    return temp;
}

//指定collection的contentSize
- (CGSize)collectionViewContentSize{
    //内容宽度指定为collectionView的宽度(横向不发生滚动)
    CGFloat width = self.collectionView.frame.size.width;
    //取出最长的列，将其高度定位长度
    CGFloat height = [self getLongColum];
    return CGSizeMake(width, height);
}

- (CGFloat)getLongColum{
    //记录当前最长的列号
    __block NSInteger currentColum = 0;
    //假设最长的列高度为0
    __block CGFloat longHeight = 0;
    //枚举数组中的元素
    [_columHeightArr enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj floatValue] > longHeight) {
            longHeight = [obj floatValue];
            currentColum = idx;
        }
    }];
    return longHeight + _insetSpace.bottom;
}

//取得最短的列
- (NSInteger)getShortColum{
    //记录当前最短的列号
    __block NSInteger currentColum = 0;
    //假设最短的列高度为float的最大值
    __block CGFloat shortHeight = MAXFLOAT;
    //枚举数组中的元素
    [_columHeightArr enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj floatValue] < shortHeight) {
            shortHeight = [obj floatValue];
            currentColum = idx;
        }
    }];
    return currentColum;
}

@end
```
