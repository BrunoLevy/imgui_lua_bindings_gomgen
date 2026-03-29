# This is a rewrite of ImGui lua bindings

It is forked from [this repository](https://github.com/casssoft/imgui_lua_bindings), that
initially used a perl script to parse `imgui.h`. Since it (unfortunately) does not work any
more with more recent versions of Dear Imgui, I adapted the code generator of
[Graphite](https://github.com/BrunoLevy/GraphiteThree) and created a new backend for imgui/lua.
It is moslty compatible with the original binding, but note that there is a couple of changes:
- enum constants added to the global namespace instead of creating a namespace for each enum, which may
  seem ugly, but then Lua code resembles C++ code more
- there are two different files / initialization functions for ImGui
  and ImDrawList, so that one can bind what's needed (and just what's
  needed)
- the initialization functions take a `lua_State*`
- the ImDrawList bindings have the same name of their C++ counterparts
  and take an `ImDrawList*` as the first parameter

What follows is the README that came with the original bindings
(updated to take into account the list of changes above).

# These are imgui bindings for lua.

ImGui https://github.com/ocornut/imgui

These bindings support a lot of common imgui operations except for initializing imgui.

This repo only deals with binding ImGui with lua and doesn't deal with setting up the ImGui impl files required to run ImGui, check out the ImGui repo for help with that.

For LOVE bindings check out https://github.com/slages/love-imgui (uses these C++ bindings and does the rest of the work for you).

Function support for dear imgui 1.92:

    Normal Imgui functions:   Supported: 314 Unsupported: 61
    Imgui DrawList functions: Supported: 52 Unsupported: 16


## How to call these imgui bindings from lua

It mostly is the same as calling from C++ except for dealing with pointers and ImVecs

Function definition in C++
```c++
    IMGUI_API bool RadioButton(const char* label, bool active);
```
How to call function in lua

```lua
ret = imgui.RadioButton("String goes here", isActive)
```

## Pointers:

Lua doesn't have pointers but it has multiple return values
so instead of giving it a pointer, you give it a value and it will pass
back the new value.

Function definition in C++
```c++
    IMGUI_API void ShowTestWindow(bool* opened = NULL);
```
How to call function in lua
```lua
opened = imgui.ShowTestWindow(opened)
```
This can make some functions a bit weird. For example Begin.

Function definition in C++
```c++
    IMGUI_API bool Begin(const char* name, bool* p_opened = NULL, ImGuiWindowFlags flags = 0);
```

How to call function in lua (Note: optional arguments still work)
```lua
shoulddraw, p_opened = imgui.Begin("Name", p_opened)
```


Begin normally returns whether it's desirable to draw or not as well as setting the value
of p_opened to whether the window is open or not. Still use the first return value to decide to whether to call the ImGui functions for that window and imgui.End().


## ImVecs:

Those are arguments are expanded to separate variables instead of one object.

Function definition in C++
```c+++
    IMGUI_API void SetNextWindowPos(const ImVec2& pos, ImGuiSetCond cond = 0);
```

How to call function in lua
```lua
imgui.SetNextWindowPos(100, 50)
```

## DrawList functions:

~All functions that operate on drawlists are called with the prefix DrawList~

Function definition in C++
```c++
    IMGUI_API void AddLine(const ImVec2& a, const ImVec2& b, ImU32 col, float thickness = 1.0f);
```

How to call function in lua
```lua
imgui.AddLine(list, minX, minY, maxX, maxY, 0xFF0000FF, 2)
```
where `list` is for instance obtained using `imgui.GetWindowDrawList(), imgui.GetBackgroundDrawList(), imgui.GetForegroundDrawList()`

Note you must specifiy the color in hex:
`0x(ALPHA)(BLUE)(GREEN)(RED)`
`0xFF0000FF` = full opacity red


## Enums:

~Enums are exposed through a "constant" table. They're namespaced with "ImGui" stripped from the name.~
Enum constants are added to the global namespace, using the same name as their C++ counterparts.


```c++
ImGui::SetNextWindowSize(ImVec2(550,680), ImGuiSetCond_FirstUseEver);
ImGui::Begin("Demo", p_open, ImGuiWindowFlags_ShowBorders);
ImGui::End()
```

```lua
imgui.SetNextWindowSize(550,680, ImGuiSetCond_FirstUseEver)
imgui.Begin("Demo", true, ImGuiWindowFlags_ShowBorders)
imgui.End()
```

## How to build:
```
g++ -c imgui_lua_bindings.cpp
g++ -c imdrawlist_lua_bindings.cpp
```
(supposing that `imgui.h` and `lua.h` are already in the include path)

## How to initialize from your C++ code:
```c++
extern void LoadImGuiBindings(lua_State* L);
extern void LoadImDrawListBindings(lua_State* L);
...
main() {
  ...
  lua_State* L = ...
  LoadImGuiBindings(L);
  LoadImDrawListBindings(L);
}
```

## How to re-generate `imgui_lua_bindings.cpp` and `imdrawlist_lua_bindings.cpp`
You will need to do that if for instance you want bindings for a different version of dear imgui.
The bindings are generated using `gomgen`, that is part of the Graphite software.

1. get and compile Graphite [here](https://github.com/BrunoLevy/GraphiteThree/wiki#installing)
2. add Graphite's binaries directory to your `PATH`
3. generate bindings:
```
gomgen -oimgui_lua_bindings.cpp -iimgui.h -sImGui -DIMGUI_DISABLE_OBSOLETE_FUNCTIONS -lua
gomgen -oimdrawlist_lua_bindings.cpp -iimgui.h -sImDrawList -DIMGUI_DISABLE_OBSOLETE_FUNCTIONS -lua
4. make sure you take the latest version of `luawrap_runtime.h` from `GraphiteThree/src/bin/gomgen/`
```

## License

This version: BSD 3 clauses

Original version:
I don't feel like writing a license so here's it in laymans terms...

You can use this code for whatever just don't redistribute the exact same source code and try to sell it, or claim that the source code was made by you.
You can compile this source code and sell it. You can change this source code and sell the modified version.
You can include this source code in whatever open source project (let me know please!). You can include it in whatever closed source project.

Just be chill and if you make a billion dollars send me an email or something.

## Contributing
If you have any improvements create a pull request! If you want a function supported or disagree with how the bindings work make an issue!
