---
title: OC-Block使用和分析
date: 2016-08-06 21:14:52
tags: [oc,block]
categories: iOS
---
#block使用

##第一部分

###定义和使用Block，

```
void (^printBlock)() = ^(){
    printf("no number");
};
printBlock();

printBlock(9);
    
    
int mutiplier = 7;
//（3）定义名为myBlock的代码块，返回值类型为int
int (^myBlock)(int) = ^(int num){
  	return num*mutiplier;
}

//使用定义的myBlock
int newMutiplier = myBlock(3);
printf("newMutiplier is %d",myBlock(3));
    

```

###定义在-viewDidLoad方法外部

```
//（2）定义一个有参数，没有返回值的Block
void (^printNumBlock)(int) = ^(int num){
    printf("int number is %d",num);
};

```

定义Block变量，就相当于定义了一个函数。但是区别也很明显，因为函数肯定是在-viewDidLoad方法外面定义，而Block变量定义在了viewDidLoad方法内部。当然，我们也可以把Block定义在-viewDidLoad方法外部，例如上面的代码块printNumBlock的定义，就在-viewDidLoad外面。

再来看看上面代码运行的顺序问题，以第（3）个myBlock距离来说，在定义的地方，并不会执行Block{}内部的代码，而在myBlock(3)调用之后才会执行其中的代码，这跟函数的理解其实差不多，就是只要在调用Block（函数）的时候才会执行Block体内（函数体内）的代码。所以上面的简单代码示例，我可以作出如下的结论，

######（1）在类中，定义一个Block变量，就像定义一个函数；

######（2）Block可以定义在方法内部，也可以定义在方法外部；

######（3）只有调用Block时候，才会执行其{}体内的代码；

######（PS：关于第（2）条，定义在方法外部的Block，其实就是文件级别的全局变量）

那么在类中定义一个Block，特别是在-viewDidLoad方法体内定义一个Block到底有什么意义呢？我表示这时候只把它当做私有函数就可以了。我之前说过，Block其实就相当于代理，那么这时候我该怎样将其与代理类比以了解呢。这时候我可以这样说：本类中的Block就相当于类自己服从某个协议，然后让自己代理自己去做某个事情。

##第二部分

###__block关键字的使用

在Block的{}体内，是不可以对外面的变量进行更改的，比如下面的语句，

```
int x = 100;
void (^sumXAndYBlock)(int) = ^(int y){
x = x+y;
printf("new x value is %d",x);
};
sumXAndYBlock(50);

```

这段代码有什么问题呢，Xcode会提示x变量错误信息：Variable is not assigning (missing __block type)，这时候给int x = 100;语句前面加上__block关键字即可，如下，

```
__block int x = 100;

```
这样在Block的{}体内，就可以修改外部变量了。

##第三部分：Block作为property属性实现页面之间传值

第二个页面总的代码如下

```
@interface NextViewController : UIViewController
@property (nonatomic, copy) void (^NextViewControllerBlock)(NSString *tfText);
 
@end
//NextViewContorller.m 文件
- (IBAction)popBtnClicked:(id)sender {
    if (self.NextViewControllerBlock) {
        self.NextViewControllerBlock(self.inputTF.text);
    }
    [self.navigationController popViewControllerAnimated:YES];
}

```
第一个页面总的代码如下

```
- (IBAction)btnClicked:(id)sender {
    NextViewController *nextVC = [[NextViewController alloc] initWithNibName:@"NextViewController" bundle:nil];
    nextVC.NextViewControllerBlock = ^(NSString *tfText){
        [self resetLabel:tfText];
    };
    [self.navigationController pushViewController:nextVC animated:YES];
}
#pragma mark - NextViewControllerBlock method
- (void)resetLabel:(NSString *)textStr
{
    self.nextVCInfoLabel.text = textStr;
}

```

#block使用分析 就像delegate的简化版
代理设计模式对于iOS开发的人来说肯定很熟悉了，代理delegate就是委托另一个对象来帮忙完成一件事情，为什么要委托别人来做呢，这其实是MVC设计模式中的模块分工问题，例如View对象它只负责显示界面，而不需要进行数据的管理，数据的管理和逻辑是Controller的责任，所以此时View就应该将这个功能委托给Controller去实现，当然你作为码农强行让View处理数据逻辑的任务，也不是不行，只是这就违背了MVC设计模式，项目小还好，随着功能的扩展，我们就会发现越写越难写；还有一种情况，就是这件事情做不到，只能委托给其他对象来做了，下面的例子中我会说明这种情况。

##实际情景使用


下面的代码我想实现一个简单的功能，场景描述如下：TableView上面有多个CustomTableViewCell，cell上面显示的是文字信息和一个详情Button，点击button以后push到一个新的页面。为什么说这个场景用到了代理delegate？因为button是在自定义的CustomTableViewCell上面，而cell没有能力实现push的功能，因为push到新页面的代码是这样的，

```
[self.navigationController pushViewController];

```

所以这时候CustomTableViewCell就要委托它所在的Controller去做这件事情了。

###为了方便比较  先使用delegate的方式实现
按照我的编码习惯，我喜欢把委托的协议写在提出委托申请的类的头文件里面，现在的场景中是CustomTableViewCell提出了委托申请，下面是简单的代码

```
@protocol CustomCellDelegate <NSObject>

- (void)pushToNewPage;

@end

@interface CustomTableViewCell : UITableViewCell

@property (nonatomic, assign) id<CustomCellDelegate> delegate;


@property (nonatomic, strong) UILabel *text1Label;

@property (nonatomic, strong) UIButton *detailBtn;

@end

```

上面的代码在CustomTableViewCell.h中定义了一个协议CustomCellDelegate，它有一个需要实现的pushToNewPage方法，然后还要写一个属性修饰符为assign、名为delegate的property，之所以使用assign是因为这涉及到内存管理的东西，以后的博客中我会专门说明原因。

接下来在CustomTableViewCell.m中编写Button点击代码，


[self.detailBtn addTarget:self action:@selector(btnClicked:) forControlEvents:UIControlEventTouchUpInside];

对应的btnClicked方法如下

```
- (void)btnClicked:(UIButton *)btn {
    if (self.delegate && [self.delegaterespondsToSelector:@selector(pushToNewPage)]) {
        [self.delegate pushToNewPage];
    }
}

```

上面代码中的判断条件最好是写上，因为这是判断self.delegate是否为空，以及实现CustomCellDelegate协议的Controller是否也实现了其中的pushToNewPage方法。

接下来就是受到委托申请的类，这里是对应CustomTableViewCell所在的ViewController，它首先要实现CustomCellDelegate协议，然后要实现其中的pushToNewPage方法，还有一点不能忘记的就是要设置CustomTableViewCell对象cell的delegate等于self，很多情况下可能忘了写cell.delegate = self;导致遇到问题不知云里雾里。下面的关键代码都是在ViewController.m中，

首先是服从CumtomCellDelegate协议，这个大家肯定都知道，就像很多系统的协议，例如UIAlertViewDelegate、UITextFieldDelegate、UITableViewDelegate、UITableViewDatasource一样。

```
@interface ViewController ()<CustomCellDelegate>

@property (nonatomic, strong) NSArray *textArray;

@end

//然后是实现CustomCellDelegate协议中的pushToNewPage方法，

- (void)pushToNewPage {
    DetailViewController*detailVC = [[DetailViewController alloc] init];
    [self.navigationController pushViewController:detailVC animated:YES];
}

//还有一个步骤最容易被忘记，就是设置CumtomTableViewCell对象cell的delegate，如下代码，

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *simpleIdentify = @"CustomCellIdentify";
    CustomTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:simpleIdentify];
    if (cell == nil) {
        cell = [[CustomTableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:simpleIdentify];
    }
    //下面代码很关键
    cell.delegate = self;

    cell.text1Label.text = [self.textArray objectAtIndex:indexPath.row];

    return cell;
}

```

通过cell.delegate = self;确保了CustomTableViewCell.m的判断语句if(self.delegate && ...){}中得self.delegate不为空，此时的self.delegate其实就是ViewController，cell对象委托了ViewController实现pushToNewPage方法。这个简单的场景描述了使用代理的一种情况，就是CustomTableViewCell没有能力实现pushViewController的功能，所以委托ViewController来实现。

###下面使用block的方式实现

Block是一个C语言的特性，它就是C语言的函数指针，在使用中最多的就是进行函数回调或者事件传递，比如发送数据到服务器，等待服务器反馈是成功还是失败，此时block就派上用场了，这个功能的实现也可用使用代理，这么说的话，感觉block是不是有点像代理了呢？

我之前接触block，都是使用它作为函数参数，当时感觉不是很理解。现在在项目中，很多时候block作为property，这样更加简单直接，想想，其实property不就是定义的合成存储的变量嘛，而block作为函数参数也是定义的变量，所以作为函数参数或者作为property本质没有区别。

看一看别人总结的block的语法吧，http://fuckingblocksyntax.com，这个链接亮了，fucking block syntax，操蛋的block语法啊。block有如下几种使用情况，

1、作为一个本地变量（local variable）

```
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {
	//blablabla
};
```

2、作为@property

```
@property (nonatomic, copy) returnType (^blockName)(parameterTypes);

```

3、作为方法的参数（method parameter）

```
- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;

```

4、作为方法参数的时候被调用

```
[someObject someMethodThatTakesABlock: ^returnType (parameters) {...}];

```
5、使用typedef来定义block，可以事半功倍

```
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^returnType(parameters) {...};

```

上面我也只是复制粘贴了一下，接下来还是实现点击CustomTableViewCell上面的Button实现页面跳转的功能，我之前不止一次的类比block就像delegate，这边我也是思维惯性，下面的内容我就当block为代理，一些用词描述还是跟delegate差不多。首先，在提出委托申请的CustomTableViewCell中定义block的property，

```
@interface CustomTableViewCell : UITableViewCell

@property (nonatomic, strong) UILabel *text1Label;

@property (nonatomic, strong) UIButton *detailBtn;
```

```
/*delegate的定义 我没有删除，因为大家可以类比了看下*/

@property (nonatomic, assign) id<CustomCellDelegate> delegate;

/*这里定义了ButtonBlock*/

@property (nonatomic, copy) void (^ButtonBlock)();

@end
```

这里用copy属性来修饰ButtonBlock property，这个原因，我会在以后的博客中作专门的解释。

接下来在CustomTableViewCell中给它上面的detailBtn绑定点击方法，

```
[self.detailBtn addTarget:self action:@selector(btnClicked:) forControlEvents:UIControlEventTouchUpInside];

- (void)btnClicked:(UIButton *)btn {
    //这是之前的delegate
    if (self.delegate && [self.delegate respondsToSelector:@selector(pushToNewPage)]) {
        [self.delegate pushToNewPage];
    }
    
    //这是现在我们要说的block    
    if (ButtonBlock) {
        ButtonBlock();
    }
}

```

下面是一个关键性的地方，在ViewController2中设置其CustomTableViewCell的cell对象的ButtonBlock，也就是给它赋值，此处我还是保留了cell.delegate = self;  ##代码##

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    NSString *blockIdentify = @"BlockIdentify";
    CustomTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:blockIdentify];
    if (cell == nil) {
        cell = [[CustomTableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:blockIdentify];
    }
    cell.text1Label.text = [self.textArray objectAtIndex:indexPath.row];
    //delegate的不可缺少的代码，这里放在这儿只是为了给各位类比一下
    cell.delegate = self;
    
    //ButtonBlock不可缺少的代码
    cell.ButtonBlock = ^{
        [self pushToNewPage2];
    };
    return cell;
}

```

之所以cell.ButtonBlock = ^{};赋值，是因为我们我们是这样定义ButtonBlock的，void (^ButtonBLock)()，表示无返回值无参数。


你们看这个方法是不是与CustomCellDelegate协议中的pushToNewPage方法类似。然后在回过头来类比一样，是不是block就是精简版的delegate，因为delegate设计模式要写协议CustomCellDelegate，还有容易遗漏cell.delegate = self;但是block使用的时候就简单多了。