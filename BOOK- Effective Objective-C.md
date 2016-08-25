#Effective Objective-C

###Guide
1. [熟悉Objective-C]()
	1. [了解Objective-C语言的起源]()
	2. [在类的头文件中尽量少引入其它头文件]()
	3. [多用字面量语法，少用与之等价的方法]()
	4. [多用类型常量，少用#define预处理指令]()
	5. [用枚举表示状态、选项、状态码]()
	6. []()
2. [对象、消息、运行期]()
	1. [理解“属性”这一概念]()
	2. [在对象内部尽量直接访问实例变量]()
	3. [理解“对象等同性”这一概念]()
	4. [以“类族模式”隐藏实现细节]()
	5. [在既有类中使用关联对象存放自定义数据]()
	6. [理解objc_msgSend的作用]()
	7. [理解消息转发机制]()
	8. [用“方法调配技术”调试“黑盒方式”]()
	9. [理解“类对象”的用意]()
3. [接口与API设计]()
	1. [用前缀避免命名空间冲突]()
	2. [提供“全能初始化方法”]()
	3. [实现description方法]()
	4. [尽量使用不可变对象]()
	5. [使用清晰而协调的命名方式]()
	6. [为私有方法名加前缀]()
	7. [理解Objective-C错误模型]()
	8. [理解NSCopying协议]()
4. [协议与分类]()
	1. [通过委托与数据协议进行对象间通信]()
	2. [将类的实现代码分散到便于管理的数个分类之中]()
	3. [总是为第三方类的分类名称加前缀]()
	4. [勿在分类中声明属性]()
	5. [使用“class-continuation分类”隐藏实现细节]()
	6. [通过协议提供匿名对象]()
5. [内存管理]()
	1. [理解引用计数]()
	2. [用ARC简化引用计数]()
	3. [在dealloc方法中只释放引用并解除监听]()
	4. [编写“异常安全代码”时留意内存管理问题]()
	5. [以弱引用避免保留环]()
	6. [以“自动释放池块”降低内存峰值]()
	7. [用“僵尸对象”调试内存管理问题]()
	8. [不要使用retainCount]()
6. [块与大中枢派发]()
	1. []()
	2. []()
	3. []()
	4. []()
	5. []()
	6. []()
	7. []()
	8. []()
	9. []()
	10. []()
7. [系统框架]()

##熟悉Objective-C
###了解Objective-C语言的起源
1. Objective-C使用消息结构而非函数调用。Objective-C由Smalltalk（消息型语言的鼻祖）演化。使用消息结构的语言，其运行时所应执行的代码由其运行环境决定；而是用函数调用的语言，由编译器决定。如果代码中调用的函数是多态的，那么在运行时就需要查询虚方法表（virtual table）来决定执行哪个函数，但是使用消息结构的语言，无论是否多态，都会在运行时去查找所需执行的方法。编译器甚至不关心接收消息的对象，接收消息的对象问题也在运行时决定（称之为动态绑定dynamic binding）。
2. Objective-C的重要工作在运行期组件（runtime component）中完成，而非编译器，使用Objecitve-C的面向对象特性的全部数据结构和函数都在运行期组件中。只需要更新运行期组件，便可以获得性能的提升，而对于重要工作在编译器完成的语言，如果想获得类似的性能提升，则需要重新编译代码。
3. 对象总是分配在堆空间中的，绝不会分配在栈空间中（栈中的是指针）。分配在堆中的内存需要直接管理，而在栈上的内存随栈帧弹出而自动清理。

###在类的头文件中尽量少引入其它头文件
1. 将引入头文件的时间尽量延后，可以减少编译时间。
2. 循环引用的头文件不会导致死循环，但是会导致两个类中有一个无法被正确编译。
	
	```
	//
	//  A.h
	//  EffectiveObjectiveCTest
	//
	//  Created by 黄伟平 on 16/8/24.
	//  Copyright © 2016年 黄伟平. All rights reserved.
	//
	
	#import <UIKit/UIKit.h>
	#import "B.h"
	
	@interface A : NSObject
	
	@property (nonatomic, strong) B *b;
	
	@end

	```
	```
	//
	//  B.h
	//  EffectiveObjectiveCTest
	//
	//  Created by 黄伟平 on 16/8/24.
	//  Copyright © 2016年 黄伟平. All rights reserved.
	//
	
	#import <UIKit/UIKit.h>
	#import "A.h"
	
	@interface B : NSObject
	
	@property (nonatomic, strong) A *a;
	
	@end

	```
3. 如果要声明类遵从某个协议，那就不能用向前声明来引入协议，向前声明只能告诉编译器有某个协议，但是编译器必须还要知道协议中的方法。
4. 应该尽量把头文件放在class-continuation分类中，可以减少编译时间、降低彼此依赖的程度。

###多用字面量语法，少用与之等价的方法
1. ```NSArray *languages = @[@"Objective-C", @"Java"];```

###多用类型常量，少用#define预处理指令
1. 对比
	
	```
	#define ANIMATION_DURATION 0.3
	```
	```
	static const NSTimeInterval kAnimationDuration = 0.3;
	```
	类型常量能够清楚地描述常量的含义（NSTimeInterval）。
2. 如果常量局限于某个编译单元（translation unit，也就是实现文件，implementation file）之内，那么在前面加k；若在类之外可见，用类名作为前缀。
3. 用static表示变量仅在此变量的编译单元内可见。
4. 用const来避免常量被修改。
5. 如果需要对外公开常量，应该：
	
	```
	// .h
	extern NSString *const EOCStringConstant;
	```
	```
	// .m
	NSString *const EOCStringConstant = @"value";
	```
	常量定义可以从右向左解读。

###用枚举表示状态、选项、状态码
1. 凡是需要以按位或操作来实现的枚举都使用NS_OPTIONS，如果枚举不可以互相组合，应使用NS_ENUM。
2. 如果用switch来进行枚举分支，别写default，这样如果之后需要加入新的枚举值，就可以根据报错，一处不漏地修改所有相关switch。

##对象、消息、运行期
###理解“属性”这一概念
1. 如果代码使用编译器计算出来的偏移量，那么在修改了类的定义之后，就必须重新编译。比如某个代码库中使用一份旧的类定义，如果和其相链接的代码使用了新的类定义，那么运行时就会出现不兼容现象（incompatibility）。Objective-C为了解决这个问题，将实例变量当做一种存储偏移量所用的“特殊变量”，交由类对象保管。偏移量会在运行期查找，如果类的定义变了，那么存储的偏移量也就变了。这样就可以保证，无论何时访问实例变量，总能使用正确的偏移量。甚至可以在运行期向类中新增实例变量，这就是稳固的“应用程序二进制接口”（Application Binary Interface， ABI）。
2. 编译器会把点语法转换为存取方法。
3. @dynamic关键字告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法。而且，在编译访问属性的代码时，即使编译器发现没有定义存取方法，也不要报错，要相信，这些方法可以在运行期找到。
4. unsafe_unretained的目标对象被摧毁时，属性值并不会自动设为nil。
5. 开发iOS程序时，一般都会用nonatomic，因为用atomic会有性能问题；而开发Mac OS时，用atomic一般不会有性能问题。

###在对象内部尽量直接访问实例变量
1. 对比
	1. 不经过Objective-C的方法派发（method dispatch）来访问实例变量，速度快。编译器会生成的代码会直接访问保存对象实例变量的内存。
	2. 直接访问不会调用其“设置方法”，绕过了内存管理语义。比如在ARC下直接访问一个copy变量，并不会拷贝该属性，只会保留新值，释放旧值。
	3. 直接访问不会出发KVO。
	4. 通过属性来访问可以方便加入断点。
2. 写入实例变量时，可以用设置方法，读取时可以直接读取。
3. 注意子类有可能覆写了设置方法。
4. 如果属性是惰性初始化， 那就必须用获取方法来访问属性。

###理解“对象等同性”这一概念
1. 判断等同性的关键方法：
	
	```
	- (BOOL)isEqual:(id)object;
	- (NSUInteger)hash;
	```
	当且仅当两个对象的内存地址相等，它们才完全相等。如果```isEqual```方法判定两个对象相等，那么```hash```方法返回的值也一样；如果```hash```返回的值一样，```isEqual```未必认为两个对象相等。
2. ```isEqual```例子

	```
	- (BOOL)isEqual:(id)object
	{
		if (self == object) return YES;
		if ([self class] != [object class]) return NO;
		
		EOCPerson *otherPerson = (EOCPerson *)object;
		if (![_firstName isEqualToString:otherPerson.firstName]) return NO;
		if (![_lastName isEqualToString:otherPerson.lastName]) return NO;
		
		return YES;
	}
	```
3. ```hash```例子
	
	```
	- (NSUInteger)hash
	{
		NSUInteger firstNameHash = [_firstName hash];
		NSUInteger lastNameHash = [_lastName hash];
		return firstNameHash ^ lastNameHash;
	}
	```
	这样计算hash值既能保持高效率，又能使hash值在一定范围内。编写hash方法时，需要在碰撞和运算复杂度之间取舍。
4. 如果经常需要判断等同性，可以自己写一个特定类的等同性判断方法。这样可以不用检验传入参数的类型，提高效率。

	```
	- (BOOL)isEqualToPerson:(EOCPerson *)otherPerson
	{
		if (self == otherPerson) return YES;
		
		if (![_firstName isEqualToString:otherPerson.firstName]) return NO;
		if (![_lastName isEqualToString:otherPerson.lastName]) return NO;
		
		return YES;

	}
	```
	也应一并改写```isEqual```方法，常用写法如下：
	
	```
	- (BOOL)isEqual:(id)object
	{
		if ([self class] == [object class])
		{
			return [self isEqualToPerson:(EOCPerson *)object];
		}
		else
		{
			return [super isEqual:object];
		}
	}
	```
5. 在容器中放入可变类对象的时候，不要再改变其hash值。

###以“类族模式”隐藏实现细节
1. 比如```+ (UIButton *)buttonWithType:(UIButtonType)type```，这个工厂方法返回的类型取决于传入参数，但是这些返回的类型都继承自```UIButton```。
2. 像NSArray这样的类的背后其实是一个类族（对于大多数collection来说都是这样），比如以下的代码是错误的：

	```
	id maybeAnArray = ...;
	if ([maybeAnArray class] == [NSArray class])
	{
		// Will never hit
	}
	```
	因为mybeAnArray的类型绝不可能为NSArray，而是类族中的某个类型。若想判断某个类是否在某个类族中，可以用：
	
	```
	id maybeAnArray = ...;
	if ([maybeAnArray isKindOfClass:[NSArray class]])
	{
		// Will be hit
	}
	```
3. 如果希望向类族中加入新的子类，就必须知道工厂方法的源代码。但是对于Cocoa中NSArray这样的类族来说，可以通过以下方法新增子类：
	1. 子类应该继承自类族中的抽象基类。
	2. 子类应该定义自己的数据存储方式。
	3. 子类应该覆写超类文档中指明需要覆写的方法。

###在既有类中使用关联对象存放自定义数据
1. ```void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy)```
	```id objc_getAssociatedObject(id object, void *key)```
	```void objc_removeAssociatedObjects(id object)```
2. 设置关联对象用到的键是不透明指针（[opaque pointer](http://stackoverflow.com/questions/7553750/what-is-an-opaque-pointer-in-c)）。如果在两个键上调用```isEqual```方法的返回值为YES，那么NSDictionary认为两者相等；但是在设置关联对象的时候，如果希望两个键指向同一个值，那么两者必须是完全一样的指针。所以一般用静态全局变量作为键。
3. 只有在其他方法不可行的时候才用关联对象，因为这种做法通常会引入难找的bug。

###理解objc_msgSend的作用
1. ```id returnValue = [someObject messageName:parameter];```，编译器会把这种给对象发送消息的代码转换为一条标准的C语言函数调用，调用的函数就是```objc_msgSend```：
	
	```
	void objc_msgSend(id self, SEL cmd, ...)
	```
	此函数参数可变，第一个参数表示接收者，第二个参数表示选择子，后续参数是消息中的参数，其顺序不变。
2. 消息转发会在接收者的方法列表中国年寻找符合的方法，如果没有，向继承体系往上查找。如果还是找不到，就用消息转发（message forwarding）。objc_msgSend会将匹配结果放在快速映射表中，每个类都有这个缓存，方便重复相同的消息。
3. 如果某函数的最后一项操作是调用另外一个函数，那么就可以使用“尾调用优化”。编译器会生成调转到另外一个函数的指令，而非向堆栈中推入新的栈帧。

###理解消息转发机制
消息转发分为三个阶段：

1. 征询接收者，是否可以通过动态添加方法来处理当前这个“未知选择子”，又称为“动态方法解析”。

	```
	+ (BOOL)resolveInstanceMethod:(SEL)selector
	```
2. 首先，请接收者看看是否有对象可以处理这个“未知选择子”。

	```
	- (id)forwardingTargetForSelector:(SEL)selector
	```
3. 如果还是没有，运行期系统会将所有与消息相关的细节都封装到```NSInvocation```对象中，再给接收者一次最后的机会。
	
	```
	- (void)forwardingInvocation:(NSInvocation *)invocation
	```
步骤越往后，代价越大。如果在第一步就处理完，运行期系统会把这个方法缓存起来，提高效率。

###用“方法调配技术”调试“黑盒方式”
一般来说，只有调用程序的时候才需要在运行期修改方法实现，这种做法不宜滥用。

###理解“类对象”的用意
1. 每个Objective-C对象实例都是指向某块内存的指针。
2. 描述Objective-C对象所用到的数据结构定义在运行期程序库的头文件中，id本身也定义在这里：
	
	```
	typedef struct objc_object
	{
		Class isa;	
	} *id;
	```
	Class对象也定义在运行期程序库的头文件中：
	
	```
	typedef struct objc_class *Class;
	struct objc_class
	{
		Class isa;
		Class super_class;
		const char *name;
		long version;
		long info;
		long instance_size;
		struct objc_ivar_list *ivars;
		struct objc_method_list **methodLists;
		struct objc_cache *cache;
		struct objc_protocol_list *protocol;
	}
	```
	此结构体的首个变量也是isa，说明Class本身也是Objective-C对象。super_class定义了类的超类，类对象所属的类型（isa所指向的类型）是另外一个类，叫做“元类”，用来表述类对象本身所具备的元数据。“类方法”就定义在此，因为这些方法可以理解为类对象的实例方法。每个类仅有一个“类对象”，而每个“类对象”仅有一个与之相关的“元类”。
	![](https://1.bp.blogspot.com/-wy5K5memmLw/V71aFiI_mpI/AAAAAAAAAL8/FqD1O0prYgglRRHDQfRgEMghnpJfcMl-QCLcB/s1600/EOC14.png)

3. ```isMemberOfClass:```能判断对象是否为某个特定类的实例，而```isKindOfClass:```能判断出对象是否为某类或其派生类的实例：
	
	```
	NSMutableDictionary *dict = [NSMutableDictionary new];
	[dict isMemberOfClass:[NSDictionary class]];        // NO
	[dict isMemberOfClass:[NSMutableDictionary class]]; // YES
	[dict isKindOfClass:[NSDictionary class]];          // YES
	[dict isKindOfClass:[NSArray class]];               // NO
	```
4. 应该尽量用类型信息查询方法，而不应该直接比较两个类对象是否等同。因为前者可以正确处理那些使用了消息传递机制的对象。

##接口与API设计
###用前缀避免命名空间冲突
1. Apple宣称其保留使用所有的“两字母前缀”，所以我们自己选用的前缀最好是三字母。
2. 不仅是类名，应用程序中的所有名称都应该加上前缀。如果要为类增加“分类”，还要给“分类”和“分类”中的方法添加前缀。开发者可能会忽略另外一个容易引发命名冲突的地方，那就是类的实现文件中所用的纯C函数以及全局变量。
3. 如果用第三方库开发开源库，应该注意这个第三方的库也有可能被使用我们库的人重复引入，所以应该为第三方库也加上自己的前缀。

###提供“全能初始化方法”
1. 如果创建类实例的方式不止一种，那么就会有多种初始化方法，我们要选定一个全能初始化方法，令其他初始化方法调用它来进行初始化。
2. 在类继承的时候，如果子类的全能初始化方法与超类方法的名称不同，那么总是应该覆写超类的全能初始化方法。如果不想使用超类的全能初始化方法，可以在里面抛出错误。

###实现description方法
1. 可以用NSDictionaray来输出属性。

###尽量使用不可变对象
1. 若某属性仅可在对象内部修改，则在“class-continuation”分类中将其由readonly属性扩展为readwrite属性。
2. 不要把可变的collection作为属性公开，应该提供相应的方法来修改对象中的可变collection。

###使用清晰而协调的命名方式
1. ```- getCharacters:range:```这个函数命名以get开头，因为调用此方法的时候，要在参数中传入数组。从内存管理的角度来看，使用输出参数来获取返回值更好，因为内存的管理事宜应该由函数调用者管理，而非在函数内部创建数组，而后又需要调用者释放。
2. 如果方法的返回值是新创建的，那么方法名的首个词应是返回值的类型，除非前面还有修饰语，例如```localizedString```。属性的存取方法不遵循这种命名方式，因为一般认为这些方法不会创建新对象，即使有也是内部对象的拷贝。
3. 应当把参数类型的名词放在参数前面。
4. 如果方法要在当前对象上执行操作，那么就应该包含动词；若执行操作时还需要参数，则应该在动词后面加上一个或多个名词。
5. 不使用str这种简称。
6. Boolean属性应加上is前缀。如果返回的是非属性Boolean值，应该用has或is。
7. 将get前缀用于借由“输出参数”来保存返回值的方法。

###为私有方法名加前缀
1. 给私有方法的名称加上前缀以便和公有方法区分。
2. 不要用下划线做私有方法前缀，这是留给Apple的。

###理解Objective-C错误模型
1. 只有发生了可使整个应用程序崩溃的错误的时候才使用异常。
2. 在错误不那么严重的情况下，可以指派“委托方法”来处理错误，也可以将错误信息放在NSError对象中，经由“输出参数”返回给调用者。

###理解NSCopying协议
1. NSCopying协议只有一个方法```- (id)copyWithZone:(NSZone *)zone```，之所以有NSZone是因为以前开发程序的时候会将内存分为区(zone)，而对象会创建在某个区里面。但现在每个程序只有一个默认区，所以不必理会zone。
2. 对于NSArray和NSMutableArray来说：
	
	```
	[NSMutableArray copy] => NSArray
	[NSArray mutableCopy] => NSMutableArray
	```
3. Foundation框架中的所有collection在默认情况下都使用浅拷贝，也就是只拷贝对象本身而不拷贝内容。

##协议与分类
###通过委托与数据协议进行对象间通信
1. 如果要在委托对象上调用可选方法，必须提前用类型信息查询方法检查其是否可以响应某个选择子。
	
	```
	if ([_delegate respondsToSelector:@selector(someMethod)])
	{
		[_delegate someMethod];
	}
	```
2. 每次都检查是否可以响应选择子比较耗时，可以用一个缓存变量代替。

###将类的实现代码分散到便于管理的数个分类之中
1. 如果类的方法过多，可以考虑将不同逻辑的代码放在不同的“分类”中。
2. 可以创建名为Private的分类，把这种方法全都放在里面。这个分类里的方法一般只会在类或框架内部使用，无须对外公布。这样，当使用者看到private一词，便会知道自己调用了不该调用的私有方法。可以将Private分类不公开，这样有助于保护私有代码。

###总是为第三方类的分类名称加前缀
1. 将分类方法加入类这一操作是在运行时进行的，如果类中有和分类同名的方法，类中的方法将被覆盖。

###勿在分类中声明属性
1. 把封装数据所用的所有属性定义在主接口中。
2. 在“class-continuation分类”之外的其他分类之中，可以定义存取方法，但不要定义属性。

###使用“class-continuation分类”隐藏实现细节
1. “class-continuation分类”必须定义在接续的类的实现文件中，是唯一能声明实例变量的分类，而且没有特定的实现文件，其中的方法都应该定义在类的主实现文件中。“class-continuation分类”没有名字。
2. “class-continuation分类”可以将接口中声明为“只读”属性扩展为“可读写”，以便在类的内部设置值。通常不直接访问实例变量，而是通过设置访问方法，这样可以触发KVO，因为有可能有其他的类在监听修改事件。
3. 如果对象遵从的协议是私有的，则应该在“class-continuation分类”中声明。

###通过协议提供匿名对象
1. 此处的匿名对象和其他语言中的匿名对象（内联形式所创建出来的无名类）不同，比如：
	
	```
	@property (nonatomic, weak) id<EOCDelegate> delegate;
	```
	由于该属性的类型是id<EOCDelegate>，所以实际上任何类的对象都能充当这一属性，哪怕不继承自NSObject，只要遵从EOCDelegate协议即可。
	类似于在NSDictionary中的key：
	
	```
	- (void)setObject:(id)object forKey:(id<NSCopying>)key;
	```
2. 使用匿名对象来隐藏类型名称。
3. 如果具体类型不重要，重要的是对象能够响应某些方法，可以使用匿名对象。

##内存管理
###理解引用计数
1. 如果按“引用数”回溯，最终会发现一个根对象，在Mac OS X中，此对象是NSApplication；在iOS中，是UIApplication，两者都是应用程序启动时创建的单例。
2. 在alloc或者initWithInt方法的实现代码中，也许还有其他对象也保留此对象，所以，保留计数值可能大于1。引用计数的概念不是说引用计数一定是某个值，而是执行的操作递增了计数还是递减了计数。
3. 属性存取方法中的内存管理：
	
	```
	- (void)setFoo:(id)foo
	{
		[foo retain];
		[_foo release];
		_foo = foo;
	}
	```
	顺序很重要，如果还未保留新值就先释放了旧值，且两个值指向一个对象，先执行的release就会导致对象被回收。

###用ARC简化引用计数
1. ARC中不能调用以下函数：
	
	```
	retain
	release
	autorelease
	dealloc
	```
	ARC在调用这些方法的时候，并不通过普通的Objective-C消息机制，而是调用底层C函数，节省CPU周期。
2. 使用ARC时必须遵循的方法命名规则。若方法以以下词开头，则返回的对象归调用者所有：
	
	```
	alloc
	new
	copy
	mutableCopy
	```
3. ARC只负责管理Objective-C对象的内存，CoreFoundation对象不归ARC管理，开发者须自己注意CFRetain、CFRelease的使用。

###在dealloc方法中只释放引用并解除监听
1. 程序库会以开发者察觉不到的方式操作对象，所以回收对象的时机和预期的不同，绝不可自己调用dealloc方法。一旦调用过dealloc之后，对象就不再有效，后续方法均无效。
2. 虽说应该在dealloc中释放引用，但是开销较大或系统内稀缺的资源则不在此列。比如文件描述符（file descriptor）、套接字（socket）、大块内存等，都属于这种资源。不能指望dealloc方法必定会在某个时间点执行，因为可能会有无法预料的东西持有对象。正确的做法是实现另一个方法，在确认对象用完资源后调用方法来释放资源。
3. 系统并不保证每个对象的dealloc都会被执行。在极个别情况下，当应用程序终止的时候，有可能有对象没有收到dealloc消息而继续存活，当然这些对象随着应用程序的资源被回收，实际上是消亡的，不调用dealloc反而可以提高效率。如果一定要确保清除某些对象，可以在：

	```
	// Mac OS X
	- (void)applicationWillTerminate:(NSNotification *)notification
	// iOS
	- (void)applicationWillTerminate:(UIApplication *)application
	```
	方法中清除。
4. 如果需要在dealloc中确认某些资源是否被回收，可以这样写：
	
	```
	- (void)close
	{
		// clean up resource here
		_closed = YES;
	}
	
	- (void)dealloc
	{
		if (!_closed)
		{
			// NSASSERT(...)
			[self close];
		}
	}
	```
5. 调用对象dealloc方法的线程是不可确定，如果在dealloc调用了必须在某些线程中执行的方法，要记得切换线程。

###编写“异常安全代码”时留意内存管理问题
1. 捕获异常时，一定注意将try块内所创立的对象清理干净。
2. 默认情况下，ARC不生成安全处理异常所需的清理代码。

###以弱引用避免保留环

###以“自动释放池块”降低内存峰值
1. 系统会自动创建一些线程，比如说主线程或是“大中枢派发（Grand Central Dispatch，GCD）”机制中的线程，这些线程默认都有自动释放池，每次执行“事件循环”时，便会将其清空。
2. 例子
	
	```
	NSArray *databaseRecords = ...;
	NSMutableArray *people = [NSMutableArray new];
	for (NSDictionary *record in databaseRecords)
	{
		EOCPerson *person = [[EOCPerson alloc] initWithRecord:record];
		[people addObject:person];
	}
	```
	```
	NSArray *databaseRecords = ...;
	NSMutableArray *people = [NSMutableArray new];
	for (NSDictionary *record in databaseRecords)
	{
		@autoreleasepool
		{
			EOCPerson *person = [[EOCPerson alloc] initWithRecord:record];
			[people addObject:person];
		}
	}
	```

###用“僵尸对象”调试内存管理问题
1. Cocoa提供“僵尸对象（Zombie Object）”的功能，启用这个调试功能后，运行期系统会把所有已经回收的实例转化成特殊的“僵尸对象”，而不会真正回收它们。这种对象在核心内存已经无法使用，因此不会被覆写。僵尸对象收到消息后，会抛出异常，准确说明发送过来的消息、回收前的对象。
2. _NSZombile_EOCClass实际上是在运行期生成的，当首次碰到EOCClass类的对象要变成僵尸对象的时候才创建这么一个类（惰性创建，避免造成不必要的浪费）。
3. 系统之所以要给每个变为僵尸的类都创建一个对应的新类是为了在给僵尸类发消息的时候，可以得知原来的类。运行期函数objc_duplicateClass()会把整个_NSZombie_类结构拷贝，并赋予新的名字。

###不要使用retainCount
1. 这个方法返回的保留计数值是在某个给定时间点上的值，它并没有考虑到系统会稍后把自动释放池清空，因为不会把后续释放操作从返回值中减去。所以它的值未必能真实反映实际的保留计数。
2. retainCount可能永远不返回0，因为有时系统会优化对象的释放行为，在保留计数还是1的时候就将其回收。只有在系统不进行这种优化的时候，计数值才会递减到0。

##块与大中枢派发
###理解“块”这一概念
![](http://img.blog.csdn.net/20160825005433636)
1. 在内存布局中，最重要的就是invoke变量，其为函数指针，指向块的实现代码。函数原型至少要接受一个void*型参数，此参数代表块。块是一种代替函数的语法结构，原来使用函数指针的时候，需要用“不透明的void指针”来传递状态。改用块之后，可以把原来用标准C语言特性所编写的代码封装成简明易用的接口。
2. descriptor变量指向一个结构体，每个块都包含这个结构体。其中声明了块对象的总体大小、copy和dispose两个辅助函数的指针，这两个函数用在复制、丢弃块的时候。
3. 块还会把它捕获的变量拷贝到descriptor后面，拷贝的是指向这些变量的指针。invoke函数之所以需要传递块对象为参数就是为了确认捕获的变量的内存位置。
4. 定义块的时候，所占的内存是分在栈中的。为了解决这个问题，可以用copy消息来将块复制到堆上，一旦复制到堆上，块就变成了带引用计数的对象。后续的复制操作并不会继续复制，而是增加块的引用计数。
5. 除了栈块和堆块之外，还有一类块称为“全局块”。这种块不捕获任何状态，运行时也不需要状态。块使用的内存区域可在编译期确定，全局块声明在全局内存中。全局块的复制操作是空操作，这种块相当于单例。

###为常用的块类型创建typedef
```
typedef int(^EOCSomeBlock)(BOOL flag, int value);
```
```
EOCSomeBlock block = ^(BOOL flag, int value)
{
	// do something
}
```
1. 用这种方式可以很方便地重构块，因为只要改了之后，编译器会在所有块的使用地方报错。
2. 可以同一个块定义不同的别名，用来区分用途：
	
	```
	typedef void(^ACAccountStoreSaveCompletionHandler)(BOOL success, NSError *error);
	typedef void(^ACAccountStoreAccessCompletionHandler)(BOOL granted, NSError *error);
	```

###用handler块降低代码分散程度
1. 建议在同一个块来处理成功和失败。
2. 基于handler来设计API还有个原因，就是某些代码必须运行在特定的线程上。NSNotificationCenter就属于这种API，它提供一个方法，调用者可以经由此方法来注册想要接收的通知，等到相关事件发生时，通知中心就执行注册好的块。调用者可以指定在哪个执行队列中执行块，如果没有指定，就在投递通知的线程中执行。

###用块引用其所属对象时不要出现保留环
1. 在网络数据获取器的例子中，应该等completion handler块执行完毕后，再去打破保留环，这样可以让获取器在handler块执行的时候保持存活。

###多用派发队列，少用同步锁
用串行同步队列代替同步块、锁对象的例子
	
```
_syncQueue = dispatch_queue_create("com.effectiveobjectivec.syncQueue", NULL);
	
- (NSString *)someString
{
	__block NSString *localSomeString;
	dispatch_sync(_syncQueue, ^{
		localSomeString = _someString;
	});
	return localSomeString;
}
	
- (void)setSomeString:(NSString *)someString
{
	dispatch_sync(_syncQueue, ^{
		_someString = someString;
	});
}
```
全部加锁任务都在GCD中处理，而GCD是在相当的底层实现的，可以优化到极致。  
进一步优化，设置方法不必要同步，不必等待设置完成就可以返回。
	
```
- (void)setSomeString:(NSString *)someString
{
	dispatch_async(_syncQueue, ^{
		_someString = someString;
	});
}
```
但这么写，就需要拷贝块，搞不好拷贝块的时间超过执行时间。  
如果只有读操作，就可以并行执行，我们可以利用栅栏块进一步优化
	
```
_syncQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	
- (NSString *)someString
{
	__block NSString *localSomeString;
	dispatch_sync(_syncQueue, ^{
		localSomeString = _someString;
	});
	return localSomeString;
}
	
- (void)setSomeString:(NSString *)someString
{
	dispatch_barrier_async(_syncQueue, ^{
		_someString = someString;
	});
}
```

###多用GCD，少用performSelector系列方法
1. performSelector系列方法在内存管理方面容易有疏忽，无法确定将要调用的选择子，导致ARC编译也就无法插入适当的内存管理方法。
2. performSelector系列方法所能处理的选择子太过局限，选择子的返回值类型、发送给方法的参数个数都受到限制。
3. 如果想任务放在另一个线程中，最好不要用performSelector系列方法，而是应该将任务封装到块之中，然后用GCD实现。

###掌握GCD及操作队列使用时机
1. NSOperationQueue虽然和GCD不同，但是却与之相关，开发者可以把操作以NSOperation子类的形式放在队列中，且这些操作也能并发执行。
2. GCD是纯C的API，而操作队列是Objective-C的对象。在GCD中，任务用块表示，块是轻量级的数据结构。而operation是一个重量级Objective-C对象。但是并不是所用场景都适合使用GCD。
3. NSOperation和NSOperationQueue的好处：
	1. 可以取消某个操作，直接调用cancel方法。
	2. 制定操作间的依赖关系。
	3. 可以通过KVO监测NSOperation对象属性，比如监测isCancelled或isFinished。
	4. 指定操作的优先级。

###通过Dispatch Group机制，根据系统资源状况来执行任务
通过dispatch group，可以在并发式派发队列里同时执行多项任务，此时GCD会根据系统资源状况来调度这些并发执行的任务。开发者若自己实现，则需编写大量代码。

###使用dispatch_once来执行只需运行一次线程安全代码

###不要使用dispatch_get_current_queue





