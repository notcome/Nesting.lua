#Nesting.lua

##The Problem
Redis has introduced the Lua script function for a long time. Different from a
traditional model, Lua scripts are executed as global code (like being executed
in CLI), forced to access parameters through two global arrays. This design
resulted in the impossibility of calling other scripts. Considering the
following code:

```Lua
if tonumber(ARGV[1]) <= 10 then
  redis.call('SET', KEYS[1], ARGV[2])
else
  redis.call('EVALSHA', ARGV[3], 1, KEYS[2], ARGV[1], ARGV[2])
end
```

``KEYS`` and ``ARGV`` are two global arrays. If our script call another script,
the interpreter will have to backup them and set the proper new ones. It is much
 more complicated than the current solution: disabling ``EVAL`` and ``EVALSHA``
directly.

However, this choice leads a new problem: if we want to write a fair amount of
code in Lua, how to keep the maintainability without modularization or code
reusing?

##Solve it
Nesting.lua is a Node.js project aiming solving this problem. The mechanism is
simple: making use of ``require`` function in Lua.

This function works like ``#include`` macro in C (I am not familiar with Lua so
please point them out if I make any mistakes here). Here is an example:

```Lua
-- a.lua
function double (n)
-- to make sure it works I use a wrong function name here.
  return n + n + n
end
```

```Lua
-- b.lua
require('a')
print(double(10))
```

Execute ``b.lua`` and you should see the number *30*.

Nesting.lua will imitate the ``require`` function: copying all required files to
this script (in memory of course). You could then call functions defined in
those files. Now you can wrap your self-defined command as a function and call
it from other scripts. You can also define some utility functions and make use
of them everywhere.

Here comes two problems:

1. If I want to write a Lua file with only utility functions, what should I do?

2. Redis will not execute a function. Instead, it will interpret the whole
script. If I wrap my self-defined command in one function, how to make it
compatible with Redis?

To the first problem, we could tell Nesting.lua that the file shouldn't be sent
to Redis. To the second problem, we could specify an ``return`` expression to
be added at the end of the script when it is submitted to Redis.

How to pass those information to Nesting.lua? When we pass a piece of code to
Nesting.lua interface directly, an object could be supplied. When we want to
load a directory of Lua files, we could write a JSON at the beginning of files:

```Lua
--[[
{
  "header": true,
  "return": "return this_command(KEYS, ARGV)"
}
]]--
```

You might don't like this style so you could also pass a function to parse the
Lua files and return an object.

##Usage
The project has a completely different solution at first (but much more ugly).
I changed my idea when writing this README (poor habit), so the code hasn't been
 written yet. I will finish it as soon as possible.
