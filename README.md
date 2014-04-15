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
Nesting.lua is a Node.js project aiming to solve this problem by *including*
other Lua scripts.

You could specify a list of files to be included and Nesting.lua will
automatically copy them into the script file, like ``#include`` macro in C.
Those files will be copied intactly and hence no code other than function
definitions could exist. To bypass this issue, you could provide a ``return``
expression for Redis:

```Lua
return do_something(KEYS, ARGV)
```

Now you could wrap one self-defined command as a function and call it from other
 scripts. However, sometimes we need some utility functions which won't be
invoked by Redis. To achieve this, Nesting.lua will only pass one script to
Redis if a ``return`` expression will be supplied.

How to pass those information to Nesting.lua? You could write a JSON at the
first block comment in Lua:

--[[
{
  "include" : ["utility.lua", "new_item.lua", "log.lua"],
  "return" : "return incr_visit(KEYS, ARGV)"
}
]]--

We choose this way to simplify our work. If you don't like the style, you could
customize your parser and return an object.

When you load a string as script, you could aslo provide an object directly.

##Usage
The project has a completely different solution at first (but much more ugly).
I changed my idea when writing this README (poor habit), so the code hasn't been
 written yet. I will finish it as soon as possible.
