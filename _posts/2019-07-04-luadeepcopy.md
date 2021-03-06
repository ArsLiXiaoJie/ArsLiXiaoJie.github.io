---
layout: post
title:  "lua table深拷贝"
date:   2019-07-04 19:31:01 +0800
categories: lua
tag: lua
typora-root-url: ../../arslixiaojie.github.io
---

深拷贝，只需要对table类型进行递归拷贝即可，其他类型直接用赋值拷贝或浅拷贝。

还有，拷贝后的table应与原table具有相同的元表，and元表不需要递归拷贝。



```lua
function deepcopy( src )
	local lookup_table = {}
	local function _copy( object )
		if type(object) ~= "table" then
			return object
		elseif lookup_table[object] then
			return lookup_table[object]
		end

		local new_table = {}
		lookup_table[object] = new_table
		for k,v in pairs(object) do
            --table的key可以是任意基本类型，所以不仅需要对value进行递归拷贝，也要对key进行递归拷贝
			new_table[_copy(k)] = _copy(v)
		end

        --设置元表
		return setmetatable(new_table, getmetatable(object))
	end

	return _copy(src)
end

local t = {a=3}
local tt = {
	t1 = t,
	t2 = t
}

local tcopy = deepcopy(tt)
print(t)
print(tt.t1, tt.t2)
print(tcopy.t1, tcopy.t2)

--输出
--table: 0074D710
--table: 0074D710 table: 0074D710
--table: 0074CAA0 table: 0074CAA0
--可以看到，tt里面的t1和t2其实是同一个table，所以进行深拷贝的时候要保持这种联系，而不是在深拷贝后里面变成两个独立的table，使用了lookup_table来保存已经拷贝过的对象。
--而且使用lookup_table还避免了下面这种情况

t.b = t -- 有个索引指向自己，如果不用lookup_table的话保存的话会进入死循环
local t1 = deepcopy(t)
print(t.b, t1.b)
print(t, t1)
--输出
--table: 007EAEA8 table: 007ED1B8
--table: 007EAEA8 table: 007ED1B8
```

