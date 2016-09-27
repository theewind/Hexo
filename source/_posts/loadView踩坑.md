---
title: loadView踩坑
date: 2016-09-09 10:00:31
tags: iOS
---

昨天在开始回归的时候，发现预订的订座按钮没有显示，而且big鹏还发现从NVTableView转成NVSensibleTableView也失败，于是就会调用

	方法1：
	[((NVSensibleTableView *)self.tableView) setTouchDownTarget:self selector:@selector(touchDownTableView)];
函数就会crash，这个原因是：

1. NVTableView有个setTouchDownTarget:selector:的Categroy，但是参数不一致
2. NVTableView转换NVSensibleTableView失败，即tableView根本不是NVSensibleTableView子类，所以无法调用他的setTouchDownTarget:selector:方法，所以崩溃。

<!--more-->

然后分析原因，刚开始以为是iOS10，xcode8的问题，因为之前都没有遇到这个，但是今天更新了xcode，就出现这个问题，但是因为xcode8已成事实，必须要解决，就想着是不是pod中的xib都在nova下惹的祸，于是把xib放到pod里面，也不行。然后继续看下NVTableView有没有做过修改，看了下后，发现他们有个’去除xib的‘commit，感觉就是这里出的问题。
真实原因如下：
之前NVTableViewController，有个IBOutlet的tableView属性，此tableView是从xib进行初始化的，然后继承了NVTableViewController的controller，比如BOOKOnlineBookingViewController,然后它也有个对应的BOOKOnlineBookingViewController.xib，里面有个UITableView控件，这样就把父类的tableView属性指向了子类的这个xib控件，而这个xib的控件的class是NVSensibleTableView，所以之前的[((NVSensibleTableView *)self.tableView) setTouchDownTarget:self selector:@selector(touchDownTableView)];是没有问题的，为什么这次出问题了？继续往下看
这次他做了一个这样的修改：

```
- (void)loadView {
    self.view = [[UIView alloc] initWithFrame:CGRectMake(0, 0, [UIScreen width], [UIScreen height])];
    self.view.backgroundColor = [UIColor whiteColor];
    self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    self.tableView = [[NVTableView alloc] initWithFrame:self.view.bounds style:[self tableViewStyle]];
    self.tableView.backgroundColor = [UIColor clearColor];
    self.tableView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    self.tableView.delegate = self;
    self.tableView.dataSource = self;
    [self.view addSubview:self.tableView];
}
```
重写了loadView，而且没有调用super方法，我们知道loadView的调用时机，是在第一次访问controller的`self.view==nil`的时候被调用，那么问题就来了，因为它没有调用super方法，而是自己初始化了tableView为NVTableView的实例，所以就出现前面的问题，**方法1无法调用成功**，因为这个方法是NVSensibleTableView的，而现在tableView的class是NVTableView。

我们看下laodView的说明：

>
Creates the view that the controller manages.
You should never call this method directly. The view controller calls this method when its view property is requested but is currently nil. This method loads or creates a view and assigns it to the view property.
**If the view controller has an associated nib file, this method loads the view from the nib file**. A view controller has an associated nib file if the nibName property returns a non-nil value, which occurs if the view controller was instantiated from a storyboard, if you explicitly assigned it a nib file using the initWithNibName:bundle: method, or if iOS finds a nib file in the app bundle with a name based on the view controller's class name. If the view controller does not have an associated nib file, this method creates a plain UIView object instead.
If you use Interface Builder to create your views and initialize the view controller, you must not override this method.
You can override this method in order to create your views manually. If you choose to do so, assign the root view of your view hierarchy to the view property. The views you create should be unique instances and should not be shared with any other view controller object. Your custom implementation of this method should not call super.
If you want to perform any additional initialization of your views, do so in the viewDidLoad method.
>

啥意思呢》？

**总的来说，就是loadView会先判断食否有关联的xib，如果有，就会去加载xib初始化view，如果没有，就创建一个UIView。问题就出在这里，父类NVTableViewController实现了这个方法，就等于BOOKOnlineBookingViewController写了，他没有调用super，就不会去初始化xib，自己创建了NVTableView的实例，就导致我们的xib根本没有加载。**

那么如何修改呢，我们组内基于作者的调调，修改了方法实现：

```
- (void)loadView {
    [super loadView];
    self.view.frame = CGRectMake(0, 0, [UIScreen width], [UIScreen height]);
	self.view.backgroundColor = [UIColor whiteColor];
    self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    
    if (self.tableView == nil) {
        self.tableView = [[NVTableView alloc] initWithFrame:self.view.bounds style:[self tableViewStyle]];
        self.tableView.backgroundColor = [UIColor clearColor];
        self.tableView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
        self.tableView.delegate = self;
        self.tableView.dataSource = self;
        [self.view addSubview:self.tableView];
    }
}
```
调用super方法，肯定会返回一个UIView，不管是从xib，还是自动创建的一个UIView。

ps：

1. 通过方法initWithNibName进行初始化都是`延迟加载`，在调用self.view的时候才会去真正创建view:

```
- (id)init
{
    return [self initWithNibName:@"BOOKOnlineBookingViewController" bundle:[NSBundle mainBundle]];
}
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
    ....
    }
    return self;
}
```

2. 如果实现了一个空的loadView

```
-（void）loadView {
}
```
就会导致死循环，因为viewDidLoad里面self.view==nil始终成立，就会无止境的调用loadView。接着循环继续。。。