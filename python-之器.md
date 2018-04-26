---
title: python 之器
date: 2018-04-10 08:29:05
tags:
---
# 装饰器
## 概念：
* 装饰器本质上是一个 Python 函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。  

## 代码示例：
```py
        #登陆限制装饰器
        def login_required(func):
            @wraps(func)                #wraps，可用作一个装饰器，简化调用update_wrapper的过程
            def wrapper(*args, **kwargs):
                if session.get('user_id'):
                    return func(*args, **kwargs)
                else:
                    return jsonify({
                        "message":"fail to signin"
                    }), 401
            return wrapper
```

## 引例：
> 每个人都有的内裤主要功能是用来遮羞，但是到了冬天它没法为我们防风御寒，咋办？我们想到的一个办法就是把内裤改造一下，让它变得更厚更长，这样一来，它不仅有遮羞功能，还能提供保暖，不过有个问题，这个内裤被我们改造成了长裤后，虽然还有遮羞功能，但本质上它不再是一条真正的内裤了。于是聪明的人们发明长裤，在不影响内裤的前提下，直接把长裤套在了内裤外面，这样内裤还是内裤，有了长裤后宝宝再也不冷了。装饰器就像我们这里说的长裤，在不影响内裤作用的前提下，给我们的身子提供了保暖的功效。

## 个人理解：
装饰器本质上是一个函数或者类，只不过这个函数/类比较特殊。  
1. 例如当我们需要确定某一个函数运行所需的时间时，我们可以写一个函数来完成。
```py
        import time

        def time_func():
            s_time = time.time()
            func()                  #所需要确定运行时间的函数
            e_time = time.time()
            print '运行时间：%s' % (e_time - s_time)
```

    通过以上代码可以看出，我们改变了原有函数  func  的调用方式，当程序中有很多函数需要确定运行时间时，我们如果对所有函数都进行这样的改写不仅工作量是巨大的，而且改出的bug或许是......  
2. 改用装饰器的方式我们可以写成：
```py
        def times(func):        #定义一个装饰器，可以保存在一个单独的文件中，使用时引入
            @wraps(func)
            def wrapper(*arg, **kwarg):
                s_time = time.time()
                func()
                e_time = time.time()
                print '函数耗时%s' % (e_time - s_time)
            return wrapper

        @times          #定义函数时加上装饰器, =>func = times(func)
        def func:...............
```

    可以看出正如装饰器的名字所言，装饰器就是对函数做了一个包装、修饰，但是并不改变原函数固有的功能。（python支持将函数作为参数传入另一个函数）
3. 关于 *args **kwargs:
    1. 我们所要装饰的函数有很多是需要参数的，所以说我们可能会想到在wrapper()直接添加参数，但是如果函数所需要的参数个数是不确定的，那如果每次都修改所定义的装饰器，那么装饰器便失去了其普适性。所以引入了*args **kwargs支持接受动态参数。
    2. *args表示任何多个无名参数，它是一个tuple；**kwargs表示关键字参数，它是一个dict.
4. 类装饰器:
    1. 正如前文所述装饰器也可以是类。相比于函数装饰器，类修饰器具有灵活度大、封装性等优点。使用类装饰器主要依靠类的__call__方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。
```py
            class times(object):
                def __init__(self, *args):
                    pass

                def __call__(self, func):
                    @wraps(func)
                    def wrapper(*args, **kwargs):
                        s_time = time.time()
                        self._func()
                        e_time = time.time()
                        print '函数耗时%s' % (e_time - s_time)
                    return wrapper
```

    2. 如果类装饰器有参数，则 ``__init__ ``接收参数，而 ``__call__`` 接收 ``func``

5. 装饰器的调用顺序：
    ```py
    @a
    @b
    @c
    def func():...........
    ```

    调用顺序是从里到外的，用上例而言就是      ``func = a(b(c(func)))``        
6. 装饰器的弊端及其消除：
    前文讲定义装饰器的时候没有讲到``@wraps(func)``的作用，可能会有很多疑惑。我们可以尝试将``@wraps(func)``注释掉然后查看被其装饰的函数的名称，我们会发现其名称将不会再是原函数的名称，这说明装饰器对原函数造成了影响，这是我们所不愿意见到的。所以我们使用python中的functool包中的一个装饰器来消除这个副作用。

## 装饰器route：
1. route源码：
```py
def route(self,rule,**options):

    def decorator(f):
        endpoint = options.pop('endpoints',None)
        self.add_url_rule(rule,endpoint,f,**options)
        return f
    return decorator
```

# 构造器
## ``__init__()``
1. 这里的``__init__``方法是一个特殊的方法（init是单词初始化initialization的省略形式），在使用类创建对象之后被执行，用于给新创建的对象初始化属性用。
    例如我们要创建``person``类来表示人这个物种，那么``person``表示人类全体，而``person``的每一个实例表示一个人。当我们要创建一个实例的时候我们必然要为这个实例（这个人）赋予一些属性，而``__init__``就是用来为这个实例赋予属性的，在类被初始化时调用。
2. 代码示例：
```py
        #定义一个类
        class person:
            def __init__(self, weight, height, age):
                self.weight = weight
                self.height = height
                sefl.age =age

        #实例化这个类
        Tom = person(20, 30, 40)
```

## ``__new__()``   
1. 用于对象的创建，是一个静态方法，第一个参数是``cls`` 
2. ``__new__``方法默认返回实例对象供``__init__``方法、实例方法使用。
    PS:理解不深，不做过多解释，以免产生误导。

# 解构器
## 概念：
>与构造器相对应有一个特殊的解构器（destructor）方法名为``__del__()``。然而，由于python具有垃圾对象回收机制（考引用计数），这个函数要直到该实例对象所有的引用都被清除掉后才会执行。python中的解构器是在实例释放前提供特殊处理功能的方法，它们通常没有被实现，因为实例很少被显式释放。--------摘自《python核心编程二》
## 注意事项：
1. 调用``del x``不代表调用了``x.__del__()``,它仅仅是减少了x的引用计数
2. 如果有一个循环引用或者其他原因让一个实例的引用逗留不去，该对象的``__del__()``可能永远不会被执行
3. 除非你知道你在干什么否则不要去实现``__del()__``

## 总结：
解构器这东西和我们这些小白没什么关系，还是不要用的好。