# 如何使用 Lua 实现面向对象

总的来说，就是通过重载 table 的元表中的 index 元方法，利用 Lua 的语法糖，来实现类似面向对象的用法。

实际上并不是真正的面向对象。



# 元表与元方法

元表的作用：定义原始值在特定操作下的行为

元表其实就是一个普通的表。它只是在功能上跟其他表不一样，实际上还是一张普通的表



如果我们创建一个表 `t`，然后执行 `t + 1` 这个操作，因为 tabel + 1 是不合法的行为，所以会报错

```lua
t = { a = 1 };
print(t + 1); -- 会直接报错，因为操作不合法
```

如果我们创建一个元表 `mt`， 在 `mt` 中新建一个方法 `__add`，



这样表 `t` 中的加法的元方法就被元表 `mt` 中的 `__add` 方法替换掉了，当执行 `t + 1` 的操作的时候就会调用 `__add` 方法，并返回该方法的返回值

```lua
t = { a = 1 };
mt = {
    __add = function(a, b) -- a,b为加法的左值和右值
        return a.a + b;
    end,
}
setmetatable(t, mt);

print(t + 1); -- 结果为2
```

对于 __add，它是一个元表的事件。如果任何不是数字的值（包括不能转换为数字的字符串）做加法，Lua 就会尝试调用这个元方法。首先，Lua 检查第一个操作数（即使它是合法的）， 如果这个操作数没有为 "`__add`" 事件定义元方法， Lua 就会接着检查第二个操作数。 一旦 Lua 找到了元方法， 它将把两个操作数作为参数传入元方法， 元方法的结果（调整为单个值）作为这个操作的结果。 如果找不到元方法，将抛出一个错误。



# __index



```lua
t = { a = 1 };
print(t["b"]); -- nil

mt = {
    __index = function ()
        print("this is __index in mt");
    end,
};
setmetatable(t, mt)
print(t["b"]); -- this is __index in mt
```



```lua
t = { a = 1 };
print(t["b"]); -- nil

mt = {
    __index = {
        b = 2,
        c = 3,
    },
};
setmetatable(t, mt)
print(t["b"]); -- 2
print(t["c"]); -- 3
```



# 一个语法糖

这是 Lua 支持的一种语法糖。 像 `v:name(args)` 这个样子， 被解释成 `v.name(v,args)`， 这里的 `v` 只会被求值一次。



```lua
t = {
    a = 1,
    add = function(tab, sum)
        tab.a = tab.a + sum
    end
}

t.add(t, 10)
print(t["a"]) -- 11
```

`t.add(t, 10)` 与 `t:add(10)` 这两种写法等价。t 相当于对象，他有个方法叫 add

```lua
t = {
    a = 1,
    add = function(tab, sum)
        tab.a = tab.a + sum
    end
}

t.add(t, 10)
print(t["a"]) -- 11

t:add(10)
print(t["a"]) -- 21
```



# 面向对象

实例：

实现以下需求：

- 对象名 bag
- 构造函数 new
- 装入东西 put
- 拿出东西 take
- 列出东西 list
- 清空 clear

```lua

```





[【游戏开发】在Lua中实现面向对象特性——模拟类、继承、多态-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1335538)

实现面向对象

```lua
-- 类的声明，这里声明了类名和属性，并给出了属性的初始值
Class = { x = 0, y = 0 }

-- 设置元表的索引
Class.__index = Class;

-- 构造方法（构造方法的名字是随便起的，但是习惯命名为new）
function Class:new(x, y)
    local self = {}           -- 初始化self，如果没有这句，那么所有类建立的对象如果有一个改变，其他对象都会改变
    setmetatable(self, Class) -- 将self的元表设定为Class
    self.x = x;               -- 属性初始化
    self.y = y;
    return self;              -- 返回自身
end

-- 这里定义其他类的方法
function Class:test()
    print(self.x, self.y)
end

function Class:plus()
    self.x = self.x + 1
    self.y = self.y + 1
    print(self.x, self.y)
end
```

