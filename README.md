## Object-C 中的NSProxy讲解

***

**1)用作监听对象或给对象方法追加处理**

生成一个监听类用的一个NSProxy类

```
A.h:
  @interface A : NSProxy
  // 这个用于确定需要被监听的对象
  +(id)hockObject:(id)obj;
  @end

A.m:
  @interface A ()
  { 
     // 这个值用于存储需要被监听的对象
     id self_object;
  }
  @end

  @implementation A
  // 将对象类存入自身变量中
  +(id)hockObject:(id)obj {
     A *a = [A alloc];
     a -> self_object = obj;
     return a;
  }

  -(NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
     return [self_object methodSignatureForSelector:sel];
  }
  // 在NSLog中对应位置，可以加入方法执行前后的需要操作的步骤，甚至可以修改参数(自行Google)
  -(void)forwardInvocation:(NSInvocation *)invocation {
      if ([self_object respondsToSelector:invocation.selector]) {
          NSLog(@"方法监听中：方法执行前");
         [invocation invokeWithTarget:self_object];
          NSLog(@"方法监听中：方法执行后");
      }
  }
  @end
```

实际使用

```
NSMutableArray *arr = [A hockObject:[[NSMutableArray alloc]init]];
[arr addObject:@"Hello"];
```

可变数组arr被监听，切arr的方法也被监听，上面的执行结果如下（当然，arr已经加入了@"Hello"）

```
方法监听中：方法执行前
方法监听中：方法执行后
```

***

**2)用作继承两个(或多个)对象**

生成一个继承类用的一个NSProxy类

```
@interface B : NSProxy

// 给外界的初始化接口
-(id)initWithTarget:(id)t1 target:(id)t2;

@end

```


你能实现你实现，你实现不了我实现

```
@interface B ()
{
    id target1; // 需要继承的第一个对象
    id target2; // 需要继承的第二个对象
}
@end

@implementation B

-(id)initWithTarget:(id)t1 target:(id)t2{
    target1 = t1;
    target2 = t2;
    return self;
}

-(void)forwardInvocation:(NSInvocation *)invocation {
    // 你去弄，你不行我去弄
    id target = [target1 methodSignatureForSelector:invocation.selector]?target1:target2;
    [invocation invokeWithTarget:target];
}

-(NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
   // 依旧你去弄，你不行我去弄
    NSMethodSignature *signature;
    signature = [target1 methodSignatureForSelector:sel];
    if (signature) {
        return signature;
    }
    signature = [target2 methodSignatureForSelector:sel];
    return signature;
}

-(BOOL)respondsToSelector:(SEL)aSelector
{
    // 判断能不能搞
    if ([target1 respondsToSelector:aSelector]) {
        return YES;
    }
    if ([target2 respondsToSelector:aSelector]) {
        return YES;
    }
    return NO;
}
```

完事儿之后就随随便便弄一个类搞两个就行了

```
    One *one = [[One alloc]init];
    Two *two = [[Two alloc]init];
    id b = [[B alloc]initWithTarget:one target:two];
    
    [b one];
    [b two];
```

b又能弄one又能弄two
