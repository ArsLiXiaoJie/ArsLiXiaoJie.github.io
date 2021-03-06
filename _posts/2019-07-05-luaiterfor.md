---
layout: post
title:  "lua 迭代器和泛型for"
date:   2019-07-05 15:31:01 +0800
categories: lua
tag: lua
typora-root-url: ../../arslixiaojie.github.io
---

## 迭代器和闭包

1. 迭代器可以遍历集合的每一个元素，在lua中常常使用函数来描述迭代器，每次调用该函数就返回集合的下一个元素。
2. 迭代器需要保留上一次成功调用的状态和下一次成功调用的状态，也就是它知道来自于哪里和将要前往哪里。闭包可以实现这个任务。
3. 闭包是一个内部函数，它可以访问一个或者多个外部函数的局部变量。每次闭包成功调用后这些局部变量都保存他们的值(状态)。
4. 所以一个典型的闭包结构包含两个函数：一个是闭包自己，另一个是工厂（创建闭包的函数）

```lua
--一个简单的迭代器输出table的元素
local function list_iter( t )
    --list_iter是工厂函数
	local i = 0
	local n = table.getn(t)
    --在这里，t、i、n 都属于闭包外部的局部变量
    --返回闭包函数
	return function (  )
		i = i + 1
		if i <= n then
			return t[i]
		end
	end
end

local t = {10,20,30}
local iter = list_iter(t)
while true do 
	local m = iter()
	if m == nil then
		break
	end
	print(m)
end
--输出 10 20 30
```

## 泛型for

泛型for在自己内部保存迭代函数，实际上它保存三个值：**迭代函数、状态常量、控制变量**。

```lua
for <var-list> in <exp-list> do
    <body>
end
<var-list>是以一个或多个逗号分隔的变量名列表，<exp-list>是以一个或多个逗号分隔的表达式列表，通常情况下exp-list只有一个值：迭代工厂的调用

for k, v in pairs(t) do
    print(k, v)
end
k，v 为变量列表，pairs(t)为表达式列表
```

### 泛型for的执行过程

- 初始化，计算in后面表达式的值，表达式应该返回泛型for需要的三个值：迭代函数、状态常量、控制变量；如果返回值不够三个用nil补齐，多出则忽略。
- 将状态变量和控制变量作为参数调用迭代函数（注意：对于for结构来说，状态常量没有用处，仅仅在初始化的时候获取值并传递给迭代函数）。
- 将迭代函数返回的值赋给变量列表。
- 如果返回的第一个值为nil，循环结束，否则执行循环体。
- 回到第二步再次调用迭代函数。

```lua
for var1, ..., varn in exp-list do
    block
end
等价于
do
    local _f, _s, _var = exp-list
    --_f 是迭代函数，_s 是状态常量，_var 控制变量
    while true do
        local var1, ..., varn = _f(_s, _var)
        _var = var1
        if _var == nil then
            break
        end
        block
    end
end

--简单例子
local function square( state, control )
	if state <= control then
		return
	end
	control = control + 1
	return control, control*control
end
-- 一开始 state = 9， control = 0，
-- state不变，而控制变量 control 等于square的第一个返回值，不断累加 1
for k,v in square,9,0 do
	print(k,v)
	if k == nil then
		break
	end
end
```



### pairs

```lua
function pairs(t)
    return function ( tbl, k )
    	k, v = next(tbl, k)
    	print("iter ", k,v)
    	if v then
    		return k, v
    	end
    end, t, nil
end

local tt = {}
tt[1] = 1
tt[2] = 2
tt[3] = nil
tt[4] = 4

--for k,v in pairs(tt) do
--	print(k,v)
--end

do
	local _f, _s, _var = pairs(tt) -- _f 是迭代函数，_s 是状态变量，要迭代的对象，即传入的tt，_var 是控制变量
	while true do
		local k, v = _f(_s, _var)  --执行迭代函数，返回 k,v，迭代结束时两个都是nil
		print("call iter :",k, v)
		_var = k	--刷新控制变量
		if _var == nil then
			break
		end
		print("call block :", k, v)
	end
end

--输出
--[[
iterator :      1       1
call iter :     1       1
call block :    1       1
iterator :      2       2
call iter :     2       2
call block :    2       2
iterator :      4       4
call iter :     4       4
call block :    4       4
iterator :      nil     nil
call iter :     nil     nil
]]
```



### ipairs

```lua
local tt = {}
tt[1] = 1
tt[2] = 2
tt[3] = nil
tt[4] = 4

function ipairs(t)
    return function(a, i)
        i = i + 1
        if a[i] then
            --从这里可以看出，tt[4]不会被输出了，某个下标的元素不存在就会中断
        	print("iterator : ", i, a[i])
            return i, a[i]
        end
    end, t, 0
end

-- for i,v in ipairs(tt) do
-- 	print(i,v)
-- end

do
	local _f, _s, _var = ipairs(tt)	--_f 是迭代函数，_s 是状态变量，要迭代的对象，即传入的tt，_var 是控制变量
	while true do
		local i, v = _f(_s, _var)
		print("call iter : ", i, v)
		_var = i
		if _var == nil then
			break
		end
		print("call block : ",i, v)
	end
end

--输出
--[[
iterator :      1       1
call iter :     1       1
call block :    1       1
iterator :      2       2
call iter :     2       2
call block :    2       2
call iter :     nil     nil
]]
```



### next

next 函数原型是 **next( table [ , index] )**

table 是要遍历的表

index ：next返回的值即是index索引的下一个值，当index为nil时，将返回第一个索引的值，当索引号为最后一个索引或者表为空时返回nil

常用 **next(table)** 是否返回nil来判断表是否是空表

```lua
local a = {a = "a", b = "b", c = "c", d = "d"}
local value
while next(a, value) do
    print(next(a, value))
    value = next(a, value)
end

--输出
--[[
a       a
d       d
c       c
b       b
]]
```

