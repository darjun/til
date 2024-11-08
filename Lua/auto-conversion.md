# 自动类型转换
Lua 语言在运行时提供了数值与字符串之间的自动转换（conversion）。针对字符串的所有算术操作会尝试将字符串转换为数值。Lua 语言不仅仅在算术操作时进行这种强制类型转换（coercion），还会在任何需要数值的情况下进行，例如函数 `math.sin` 的参数。

相反，当 Lua 语言发现在需要字符串的地方出现了数值时，它就会把数值转换为字符串：
```lua
print(10 .. 20)         --> 1020
```

当在数值后紧接着使用字符串连接时，必须使用空格将它们分开，否则 Lua 语言会把第一个点当成小数点。