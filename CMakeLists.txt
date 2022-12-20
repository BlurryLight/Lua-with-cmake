cmake_minimum_required(VERSION 3.11)
# default version is 5.3.6 
# redefine it use cmake -DLUA_VERSION=5.3.5 ....
# set(CMAKE_BUILD_TYPE Release)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
if(DEFINED ENV{LUA_VERSION})
    set(LUA_VERSION $ENV{LUA_VERSION}) 
else()
    set(LUA_VERSION 5.3.6) 
endif()
message("Lua Version is " ${LUA_VERSION})

# cmake中用;分割的字符串是列表
string(REPLACE "." ";" LUA_VERSION_LIST ${LUA_VERSION})
list(GET LUA_VERSION_LIST 0 LUA_VERSION_MAJOR)
list(GET LUA_VERSION_LIST 1 LUA_VERSION_MINOR)
list(GET LUA_VERSION_LIST 2 LUA_VERSION_PATCH)

# download lua
set(temp_dir ${CMAKE_CURRENT_SOURCE_DIR}/lua-${LUA_VERSION})
if(NOT EXISTS lua-${LUA_VERSION})
    set(filename lua-${LUA_VERSION}.tar.gz)
    set(lua_tgz_path ${CMAKE_CURRENT_SOURCE_DIR}/${filename})
    if(NOT EXISTS ${lua_tgz_path})
        file(DOWNLOAD
            https://www.lua.org/ftp/${filename}
            ${lua_tgz_path})
    endif()

    if(EXISTS ${temp_dir})
        file(REMOVE_RECURSE ${temp_dir})
    endif()
    file(MAKE_DIRECTORY ${temp_dir})
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar -xf ${lua_tgz_path} 
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
    # file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt.subdirectory
    #     DESTINATION ${temp_dir}/CMakeLists.txt)
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt.subdirectory ${temp_dir}/CMakeLists.txt COPYONLY)

project(lua LANGUAGES C VERSION ${LUA_VERSION})

option(LUA_SUPPORT_DL "Support dynamic loading of compiled modules" OFF)
option(LUA_BUILD_AS_CXX "Build lua as C++" OFF)
enable_language(CXX)

if(${PROJECT_NAME} STREQUAL ${CMAKE_PROJECT_NAME})
    set(TOP_LEVEL TRUE)
else()
    set(TOP_LEVEL FALSE)
endif()

option(LUA_BUILD_COMPILER "Build luac compiler" ON)
option(LUA_BUILD_BINARY "Build lua binary" $<IF:TOP_LEVEL,ON,OFF>)
option(LUA_BINARY_LINK_DYNAMIC "lua executable link dynamic lua lib" OFF)


add_subdirectory(lua-${LUA_VERSION})
# https://blog.xizhibei.me/2020/04/20/cmake-5-install-package-and-export/
# install dir
set(install_dir ${CMAKE_CURRENT_SOURCE_DIR}/${CMAKE_C_COMPILER_ID}—lua-${LUA_VERSION}-build-${CMAKE_BUILD_TYPE})
set(CMAKE_INSTALL_PREFIX ${install_dir}) # default install dir is current_dir/lua-version-build


set(Lua-headers
    lua-${LUA_VERSION}/src/lua.h
    lua-${LUA_VERSION}/src/lua.hpp
    lua-${LUA_VERSION}/src/luaconf.h
    lua-${LUA_VERSION}/src/lauxlib.h
    lua-${LUA_VERSION}/src/lualib.h
 )
install(FILES ${Lua-headers} DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(DIRECTORY lua-${LUA_VERSION}/doc  DESTINATION ${CMAKE_INSTALL_PREFIX}/doc)

install(TARGETS lua_static lua_dynamic luac
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)


if(LUA_BUILD_BINARY)
install(TARGETS lua
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
endif()
