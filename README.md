## Object-C 中的NSProxy讲解

***

**1)用作监听对象或给对象方法追加处理**

```
// 生成一个监听类用的一个NSProxy类
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
