project('luabins', ['c'])
luabins_src = files(
    'src/fwrite.c',
    'src/load.c',
    'src/luabins.c',
    'src/luainternals.c',
    'src/savebuffer.c',
    'src/save.c',
    'src/write.c',
)

deps = []

luajit_kwargs = {
    'version': '>= 2.0.0', 'default_options': ['amalgam=true', 'luajit=false', 'lua52compat=true']
}
luajit = dependency('luajit', required: get_option('system_luajit'),
                    allow_fallback: false, kwargs: luajit_kwargs)
if luajit.type_name() == 'pkgconfig'
    luajit_test = cc.run('''#include <lauxlib.h>
int main(void)
{
    lua_State *L = luaL_newstate();
    if (!L) return 1;
    // This is valid in lua 5.2, but a syntax error in 5.1
    const char testprogram[] = "function foo() while true do break return end end";
    return luaL_loadstring(L, testprogram) == LUA_ERRSYNTAX;
}''', dependencies: luajit)

    if luajit_test.returncode() == 1
        if get_option('system_luajit')
            error('System luajit found but not compiled in 5.2 mode')
        else
            message('System luajit found but not compiled in 5.2 mode')
            luajit = dependency('', required: false)
        endif
    endif
endif
if not luajit.found()
    message('Using built-in luajit')
    subproject('luajit') # Initialize subproject to ignore system luajit
    luajit = dependency('luajit', kwargs: luajit_kwargs)
    assert(luajit.type_name() == 'internal', 'System luajit used instead of built-in luajit')
endif
deps += luajit

luabins = static_library('luabins', luabins_src, dependencies: deps)
luabins_dep = declare_dependency(link_with: luabins)
