##CATextLayer

用户界面是无法从一个单独的图片里面构建的。一个设计良好的图标能够很好地表现一个按钮或控件的意图，不过你迟早都要需要一个不错的老式风格的文本标签。

如果你想在一个图层里面显示文字，完全可以借助图层代理直接将字符串使用Core Graphics写入图层的内容（这就是UILabel的精髓）。如果越过寄宿于图层的视图，直接在图层上操作，那其实相当繁琐。你要为每一个显示文字的图层创建一个能像图层代理一样工作的类，还要逻辑上判断哪个图层需要显示哪个字符串，更别提还要记录不同的字体，颜色等一系列乱七八糟的东西。

万幸的是这些都是不必要的，Core Animation提供了一个`CALayer`的子类`CATextLayer`，它以图层的形式包含了`UILabel`几乎所有的绘制特性，并且额外提供了一些新的特性。

同样，`CATextLayer`也要比`UILabel`渲染得快得多。很少有人知道在iOS 6及之前的版本，`UILabel`其实是通过WebKit来实现绘制的，这样就造成了当有很多文字的时候就会有极大的性能压力。而`CATextLayer`使用了Core text，并且渲染得非常快。

让我们来尝试用`CATextLayer`来显示一些文字。清单6.2的代码实现了这一功能，结果如图6.2所示。

清单6.2 用`CATextLayer`来实现一个`UILabel`

```objectivec
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *labelView;

@end

@implementation ViewController
- (void)viewDidLoad
{
  [super viewDidLoad];

  //create a text layer
  CATextLayer *textLayer = [CATextLayer layer];
  textLayer.frame = self.labelView.bounds;
  [self.labelView.layer addSublayer:textLayer];

  //set text attributes
  textLayer.foregroundColor = [UIColor blackColor].CGColor;
  textLayer.alignmentMode = kCAAlignmentJustified;
  textLayer.wrapped = YES;

  //choose a font
  UIFont *font = [UIFont systemFontOfSize:15];

  //set layer font
  CFStringRef fontName = (__bridge CFStringRef)font.fontName;
  CGFontRef fontRef = CGFontCreateWithFontName(fontName);
  textLayer.font = fontRef;
  textLayer.fontSize = font.pointSize;
  CGFontRelease(fontRef);

  //choose some text
  NSString *text = @"Lorem ipsum dolor sit amet, consectetur adipiscing \ elit. Quisque massa arcu, eleifend vel varius in, facilisis pulvinar \ leo. Nunc quis nunc at mauris pharetra condimentum ut ac neque. Nunc elementum, libero ut porttitor dictum, diam odio congue lacus, vel \ fringilla sapien diam at purus. Etiam suscipit pretium nunc sit amet \ lobortis";

  //set layer text
  textLayer.string = text;
}
@end
```

![图6.2](./6.2.png)

图6.2 用`CATextLayer`来显示一个纯文本标签

如果你仔细看这个文本，你会发现一个奇怪的地方：这些文本有一些像素化了。这是因为并没有以Retina的方式渲染，第二章提到了这个`contentScale`属性，用来决定图层内容应该以怎样的分辨率来渲染。`contentsScale`并不关心屏幕的拉伸因素而总是默认为1.0。如果我们想以Retina的质量来显示文字，我们就得手动地设置`CATextLayer`的`contentsScale`属性，如下：

```objectivec
textLayer.contentsScale = [UIScreen mainScreen].scale;
```

这样就解决了这个问题（如图6.3）

![图6.3](./6.3.png)

图6.3 设置`contentsScale`来匹配屏幕

`CATextLayer`的`font`属性不是一个`UIFont`类型，而是一个`CFTypeRef`类型。这样可以根据你的具体需要来决定字体属性应该是用`CGFontRef`类型还是`CTFontRef`类型（Core Text字体）。同时字体大小也是用`fontSize`属性单独设置的，因为`CTFontRef`和`CGFontRef`并不像UIFont一样包含点大小。这个例子会告诉你如何将`UIFont`转换成`CGFontRef`。

另外，`CATextLayer`的`string`属性并不是你想象的`NSString`类型，而是`id`类型。这样你既可以用`NSString`也可以用`NSAttributedString`来指定文本了（注意，`NSAttributedString`并不是`NSString`的子类）。属性化字符串是iOS用来渲染字体风格的机制，它以特定的方式来决定指定范围内的字符串的原始信息，比如字体，颜色，字重，斜体等。

###富文本

iOS 6中，Apple给`UILabel`和其他UIKit文本视图添加了直接的属性化字符串的支持，应该说这是一个很方便的特性。不过事实上从iOS3.2开始`CATextLayer`就已经支持属性化字符串了。这样的话，如果你想要支持更低版本的iOS系统，`CATextLayer`无疑是你向界面中增加富文本的好办法，而且也不用去跟复杂的Core Text打交道，也省了用`UIWebView`的麻烦。

让我们编辑一下示例使用到`NSAttributedString`（见清单6.3）.iOS 6及以上我们可以用新的`NSTextAttributeName`实例来设置我们的字符串属性，但是练习的目的是为了演示在iOS 5及以下，所以我们用了Core Text，也就是说你需要把Core Text framework添加到你的项目中。否则，编译器是无法识别属性常量的。

图6.4是代码运行结果（注意那个红色的下划线文本）

清单6.3 用NSAttributedString实现一个富文本标签。

```objectivec
#import "DrawingView.h"
#import <QuartzCore/QuartzCore.h>
#import <CoreText/CoreText.h>

@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *labelView;

@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];

  //create a text layer
  CATextLayer *textLayer = [CATextLayer layer];
  textLayer.frame = self.labelView.bounds;
  textLayer.contentsScale = [UIScreen mainScreen].scale;
  [self.labelView.layer addSublayer:textLayer];

  //set text attributes
  textLayer.alignmentMode = kCAAlignmentJustified;
  textLayer.wrapped = YES;

  //choose a font
  UIFont *font = [UIFont systemFontOfSize:15];

  //choose some text
  NSString *text = @"Lorem ipsum dolor sit amet, consectetur adipiscing \ elit. Quisque massa arcu, eleifend vel varius in, facilisis pulvinar \ leo. Nunc quis nunc at mauris pharetra condimentum ut ac neque. Nunc \ elementum, libero ut porttitor dictum, diam odio congue lacus, vel \ fringilla sapien diam at purus. Etiam suscipit pretium nunc sit amet \ lobortis";
  ￼
  //create attributed string
  NSMutableAttributedString *string = nil;
  string = [[NSMutableAttributedString alloc] initWithString:text];

  //convert UIFont to a CTFont
  CFStringRef fontName = (__bridge CFStringRef)font.fontName;
  CGFloat fontSize = font.pointSize;
  CTFontRef fontRef = CTFontCreateWithName(fontName, fontSize, NULL);

  //set text attributes
  NSDictionary *attribs = @{
    (__bridge id)kCTForegroundColorAttributeName:(__bridge id)[UIColor blackColor].CGColor,
    (__bridge id)kCTFontAttributeName: (__bridge id)fontRef
  };

  [string setAttributes:attribs range:NSMakeRange(0, [text length])];
  attribs = @{
    (__bridge id)kCTForegroundColorAttributeName: (__bridge id)[UIColor redColor].CGColor,
    (__bridge id)kCTUnderlineStyleAttributeName: @(kCTUnderlineStyleSingle),
    (__bridge id)kCTFontAttributeName: (__bridge id)fontRef
  };
  [string setAttributes:attribs range:NSMakeRange(6, 5)];

  //release the CTFont we created earlier
  CFRelease(fontRef);

  //set layer text
  textLayer.string = string;
}
@end
```

![图6.4](./6.4.png)

图6.4 用CATextLayer实现一个富文本标签。

###行距和字距

有必要提一下的是，由于绘制的实现机制不同（Core Text和WebKit），用`CATextLayer`渲染和用`UILabel`渲染出的文本行距和字距也不是不尽相同的。

二者的差异程度（由使用的字体和字符决定）总的来说挺小，但是如果你想正确的显示普通便签和`CATextLayer`就一定要记住这一点。

### `UILabel`的替代品

我们已经证实了`CATextLayer`比`UILabel`有着更好的性能表现，同时还有额外的布局选项并且在iOS 5上支持富文本。但是与一般的标签比较而言会更加繁琐一些。如果我们真的在需求一个`UILabel`的可用替代品，最好是能够在Interface Builder上创建我们的标签，而且尽可能地像一般的视图一样正常工作。

我们应该继承`UILabel`，然后添加一个子图层`CATextLayer`并重写显示文本的方法。但是仍然会有由`UILabel`的`-drawRect:`方法创建的空寄宿图。而且由于`CALayer`不支持自动缩放和自动布局，子视图并不是主动跟踪视图边界的大小，所以每次视图大小被更改，我们不得不手动更新子图层的边界。

我们真正想要的是一个用`CATextLayer`作为宿主图层的`UILabel`子类，这样就可以随着视图自动调整大小而且也没有冗余的寄宿图啦。

就像我们在第一章『图层树』讨论的一样，每一个`UIView`都是寄宿在一个`CALayer`的示例上。这个图层是由视图自动创建和管理的，那我们可以用别的图层类型替代它么？一旦被创建，我们就无法代替这个图层了。但是如果我们继承了`UIView`，那我们就可以重写`+layerClass`方法使得在创建的时候能返回一个不同的图层子类。`UIView`会在初始化的时候调用`+layerClass`方法，然后用它的返回类型来创建宿主图层。

清单6.4 演示了一个`UILabel`子类`LayerLabel`用`CATextLayer`绘制它的问题，而不是调用一般的`UILabel`使用的较慢的`-drawRect：`方法。`LayerLabel`示例既可以用代码实现，也可以在Interface Builder实现，只要把普通的标签拖入视图之中，然后设置它的类是LayerLabel就可以了。

清单6.4 使用`CATextLayer`的`UILabel`子类：`LayerLabel`

```objectivec
#import "LayerLabel.h"
#import <QuartzCore/QuartzCore.h>

@implementation LayerLabel
+ (Class)layerClass
{
  //this makes our label create a CATextLayer //instead of a regular CALayer for its backing layer
  return [CATextLayer class];
}

- (CATextLayer *)textLayer
{
  return (CATextLayer *)self.layer;
}

- (void)setUp
{
  //set defaults from UILabel settings
  self.text = self.text;
  self.textColor = self.textColor;
  self.font = self.font;

  //we should really derive these from the UILabel settings too
  //but that's complicated, so for now we'll just hard-code them
  [self textLayer].alignmentMode = kCAAlignmentJustified;
  ￼
  [self textLayer].wrapped = YES;
  [self.layer display];
}

- (id)initWithFrame:(CGRect)frame
{
  //called when creating label programmatically
  if (self = [super initWithFrame:frame]) {
    [self setUp];
  }
  return self;
}

- (void)awakeFromNib
{
  //called when creating label using Interface Builder
  [self setUp];
}

- (void)setText:(NSString *)text
{
  super.text = text;
  //set layer text
  [self textLayer].string = text;
}

- (void)setTextColor:(UIColor *)textColor
{
  super.textColor = textColor;
  //set layer text color
  [self textLayer].foregroundColor = textColor.CGColor;
}

- (void)setFont:(UIFont *)font
{
  super.font = font;
  //set layer font
  CFStringRef fontName = (__bridge CFStringRef)font.fontName;
  CGFontRef fontRef = CGFontCreateWithFontName(fontName);
  [self textLayer].font = fontRef;
  [self textLayer].fontSize = font.pointSize;
  ￼
  CGFontRelease(fontRef);
}
@end
```

如果你运行代码，你会发现文本并没有像素化，而我们也没有设置`contentsScale`属性。把`CATextLayer`作为宿主图层的另一好处就是视图自动设置了`contentsScale`属性。

在这个简单的例子中，我们只是实现了`UILabel`的一部分风格和布局属性，不过稍微再改进一下我们就可以创建一个支持`UILabel`所有功能甚至更多功能的`LayerLabel`类（你可以在一些线上的开源项目中找到）。

如果你打算支持iOS 6及以上，基于`CATextLayer`的标签可能就有有些局限性。但是总得来说，如果想在app里面充分利用`CALayer`子类，用`+layerClass`来创建基于不同图层的视图是一个简单可复用的方法。

### CATransformLayer

当我们在构造复杂的3D事物的时候，如果能够组织独立元素就太方便了。比如说，你想创造一个孩子的手臂：你就需要确定哪一部分是孩子的手腕，哪一部分是孩子的前臂，哪一部分是孩子的肘，哪一部分是孩子的上臂，哪一部分是孩子的肩膀等等。

当然是允许独立地移动每个区域的啦。以肘为指点会移动前臂和手，而不是肩膀。Core Animation图层很容易就可以让你在2D环境下做出这样的层级体系下的变换，但是3D情况下就不太可能，因为所有的图层都把他的孩子都平面化到一个场景中（第五章『变换』有提到）。

`CATransformLayer`解决了这个问题，`CATransformLayer`不同于普通的`CALayer`，因为它不能显示它自己的内容。只有当存在了一个能作用于子图层的变换它才真正存在。`CATransformLayer`并不平面化它的子图层，所以它能够用于构造一个层级的3D结构，比如我的手臂示例。

用代码创建一个手臂需要相当多的代码，所以我就演示得更简单一些吧：在第五章的立方体示例，我们将通过旋转`camara`来解决图层平面化问题而不是像立方体示例代码中用的`sublayerTransform`。这是一个非常不错的技巧，但是只能作用域单个对象上，如果你的场景包含两个立方体，那我们就不能用这个技巧单独旋转他们了。

那么，就让我们来试一试`CATransformLayer`吧，第一个问题就来了：在第五章，我们是用多个视图来构造了我们的立方体，而不是单独的图层。我们不能在不打乱已有的视图层次的前提下在一个本身不是有寄宿图的图层中放置一个寄宿图图层。我们可以创建一个新的`UIView`子类寄宿在`CATransformLayer`（用`+layerClass`方法）之上。但是，为了简化案例，我们仅仅重建了一个单独的图层，而不是使用视图。这意味着我们不能像第五章一样在立方体表面显示按钮和标签，不过我们现在也用不到这个特性。

清单6.5就是代码。我们以我们在第五章使用过的相同基本逻辑放置立方体。但是并不像以前那样直接将立方面添加到容器视图的宿主图层，我们将他们放置到一个`CATransformLayer`中创建一个独立的立方体对象，然后将两个这样的立方体放进容器中。我们随机地给立方面染色以将他们区分开来，这样就不用靠标签或是光亮来区分他们。图6.5是运行结果。

清单6.5 用`CATransformLayer`装配一个3D图层体系

```objectivec
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;

@end

@implementation ViewController

- (CALayer *)faceWithTransform:(CATransform3D)transform
{
  //create cube face layer
  CALayer *face = [CALayer layer];
  face.frame = CGRectMake(-50, -50, 100, 100);

  //apply a random color
  CGFloat red = (rand() / (double)INT_MAX);
  CGFloat green = (rand() / (double)INT_MAX);
  CGFloat blue = (rand() / (double)INT_MAX);
  face.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;

  ￼//apply the transform and return
  face.transform = transform;
  return face;
}

- (CALayer *)cubeWithTransform:(CATransform3D)transform
{
  //create cube layer
  CATransformLayer *cube = [CATransformLayer layer];

  //add cube face 1
  CATransform3D ct = CATransform3DMakeTranslation(0, 0, 50);
  [cube addSublayer:[self faceWithTransform:ct]];

  //add cube face 2
  ct = CATransform3DMakeTranslation(50, 0, 0);
  ct = CATransform3DRotate(ct, M_PI_2, 0, 1, 0);
  [cube addSublayer:[self faceWithTransform:ct]];

  //add cube face 3
  ct = CATransform3DMakeTranslation(0, -50, 0);
  ct = CATransform3DRotate(ct, M_PI_2, 1, 0, 0);
  [cube addSublayer:[self faceWithTransform:ct]];

  //add cube face 4
  ct = CATransform3DMakeTranslation(0, 50, 0);
  ct = CATransform3DRotate(ct, -M_PI_2, 1, 0, 0);
  [cube addSublayer:[self faceWithTransform:ct]];

  //add cube face 5
  ct = CATransform3DMakeTranslation(-50, 0, 0);
  ct = CATransform3DRotate(ct, -M_PI_2, 0, 1, 0);
  [cube addSublayer:[self faceWithTransform:ct]];

  //add cube face 6
  ct = CATransform3DMakeTranslation(0, 0, -50);
  ct = CATransform3DRotate(ct, M_PI, 0, 1, 0);
  [cube addSublayer:[self faceWithTransform:ct]];

  //center the cube layer within the container
  CGSize containerSize = self.containerView.bounds.size;
  cube.position = CGPointMake(containerSize.width / 2.0, containerSize.height / 2.0);

  //apply the transform and return
  cube.transform = transform;
  return cube;
}

- (void)viewDidLoad
{￼
  [super viewDidLoad];

  //set up the perspective transform
  CATransform3D pt = CATransform3DIdentity;
  pt.m34 = -1.0 / 500.0;
  self.containerView.layer.sublayerTransform = pt;

  //set up the transform for cube 1 and add it
  CATransform3D c1t = CATransform3DIdentity;
  c1t = CATransform3DTranslate(c1t, -100, 0, 0);
  CALayer *cube1 = [self cubeWithTransform:c1t];
  [self.containerView.layer addSublayer:cube1];

  //set up the transform for cube 2 and add it
  CATransform3D c2t = CATransform3DIdentity;
  c2t = CATransform3DTranslate(c2t, 100, 0, 0);
  c2t = CATransform3DRotate(c2t, -M_PI_4, 1, 0, 0);
  c2t = CATransform3DRotate(c2t, -M_PI_4, 0, 1, 0);
  CALayer *cube2 = [self cubeWithTransform:c2t];
  [self.containerView.layer addSublayer:cube2];
}
@end
```

![图6.5](./6.5.png)

图6.5 同一视角下的俩不同变换的立方体