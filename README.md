#Nesting.lua

##The Problem
Redis has introduced the Lua script function for a long time. Different from a
traditional model, Lua scripts are executed as global codes (like being executed
in CLI), which makes they access parameters through two global arrays. This
resulted in the impossibility of invoking other scripts in one script.
Considering the following code:

```Lua
if tonumber(ARGV[1]) <= 10 then
  redis.call('SET', KEYS[1], ARGV[2])
else
  redis.call('EVALSHA', ARGV[3], 1, KEYS[2], ARGV[1], ARGV[2])
end
```

Our script accesses ``KEYS`` and ``ARGV`` arrays to get the parameters. If it
calls other scripts, what will happen? One possible way is backuping the two
arrays and set new ones, which is much more complicated than the current one:
disabling ``EVAL`` and ``EVALSHA`` directly.

However, such behavior make a new problem arise: if we want to write a fair
amount of codes in Lua, how to keep the maintainability without modularization
or code reusing?

##Solve it
Nesting.lua is a Node.js project aiming solving this problem. The mechanism is
simple: making use of ``require`` function in Lua.

This function works like ``#include`` macro in C (I am not familiar with Lua so
please point it out if I make any mistakes here). Here is an example:

```Lua
-- a.lua
function double (n)
-- to make sure it works I use a bad function name here.
  return n + n + n
end
```

```Lua
-- b.lua
require('a')
print(double(10))
```

Execute ``b.lua`` and you should see a number of thirty.

Nesting.lua will expand all ``require`` function so that you could call
functions defined in other scripts. You could wrap your self-defined command as
a function and call it from other scripts. You could also define some utility
functions and make use of them everywhere.

Here comes two problems:

1. If I want to write a Lua file with only utility functions, what should I do?

2. Redis will not execute a function. It will interpret the script content. If I
 wrap all expressions in a function, how to make it compatible with Redis?

To the first problem, we could tell Nesting.lua that the file should be
processed separately. To the second problem, we could specify an expression to
be added at the end of the script when it is submitted to Redis.

How to pass those information to Nesting.lua? We could pass a piece of code to
Nesting.lua through a function with the above information. When we want to load
a directory of Lua files, we could write a JSON at the beginning at each file:

```Lua
--[[
{
  "header": true,
  "return": "return this_command(KEYS, ARGV)"
}
]]--
```

You might don't like this style so you could also pass a function to parse the
Lua files and return an ``Object``.

##Usage
The project has a completely different solution at first (but much more ugly).
I changed my idea when writing this README (poor habit), so the code hasn't been
 written yet. I will finish it as soon as possible.
