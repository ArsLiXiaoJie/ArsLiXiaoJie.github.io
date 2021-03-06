---
layout: post
title:  "lua table只读"
date:   2019-07-04 20:31:01 +0800
categories: lua
tag: lua
typora-root-url: ../../arslixiaojie.github.io
---

```lua

function read_only(inputTable)
    local travelled_tables = {}
    
    local function __read_only(tbl)
        
        if not travelled_tables[tbl] then
			--__read_only_proxy是为了避免重复创建read_only，每个table只创建一个proxy代理，在table的metatable和proxy的metatable里都设置属性__read_only_proxy，其实也是为了有互相引用的情况存在，table有索引指向自身
            local tbl_mt = getmetatable(tbl)
            if not tbl_mt then
                tbl_mt = {}
                setmetatable(tbl, tbl_mt)
            end

            local proxy = tbl_mt.__read_only_proxy
            if not proxy then
                proxy = {}
                tbl_mt.__read_only_proxy = proxy
                local proxy_mt = {
                    __index = tbl,   --对于原表tbl的访问用 __index=tbl
                    __newindex = function (t, k, v) error("error write to a read-only table with key = " .. tostring(k)) end,
                    __pairs = function (t) return pairs(tbl) end,
                    -- __ipairs = function (t) return ipairs(tbl) end,   5.3版本不需要此方法
                    --__len = function (t) return #tbl end,  --lua5.1 版本没有此元方法
                    __read_only_proxy = proxy
                }
                setmetatable(proxy, proxy_mt)
            end
            
            travelled_tables[tbl] = proxy
            --tbl_mt.__read_only_proxy 和 proxy_mt.__read_only_proxy = proxy 跟下面的递归遥相辉映
            for k, v in pairs(tbl) do
                if type(v) == "table" then
                    tbl[k] = __read_only(v)
                end
            end
        end
        return travelled_tables[tbl]
    end
    return __read_only(inputTable)
end

--[[
--加上travelled_tables是为了防止互相引用，用它来存储已经设置过proxy的table的映射，比如
local t = {a = 1, b = 2}
t.c = t -- 有索引指向本身
local tt = read_only(t)
print(tt.a, tt.b, tt.c)
print(tt.c)
print(tt.c.a, tt.c.b)
--输出
--1       2       table: 0039C6A8
--1       2
--如果不用 travelled_tables，将会进入死循环
]]


--[[
function testReadOnly( tbl )
	setmetatable(tbl, {
		__newindex = function ( tbl, key, value )
			print("can not set read only")
		end
	})
end
local t4 = {a = 1}
testReadOnly(t4)
t4.b = 2
t4.a = 4
print(t4.a, t4.b)--输出 4       nil
--之前以为可以这么设置只读，发现是不行的，对已经存在的字段可以重新赋值
]]

```

