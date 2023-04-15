# v8引擎静态库

v8引擎静态库库文件下载使用。

+ 支持ia32/x64架构 windows MSVC下 32/64位 debug/release版本
+ 支持ia32/x64架构 linux GNU下 32/64位 debug/release版本
+ 支持x64/arm架构 mac GNU下 64位 debug/release版本

版本会持续跟新。由于github不支持大文件上传，所以把文件放在了阿里云盘。下载链接在文件下方。<br>

cmake使用例子可以参考我的项目 [v8-learn](https://github.com/dengweichi/v8-learn)
 
>  每个静态库压缩包的的目录结构如下
````
|- v8.lib.${version} 
    |- include
    |- lib
       |- linux
          |- ia32
             |- debug
                |- libv8_monolith.a
                |- libv8_libplatform.a
                |- libv8_libbase.a
             |- release
                |- libv8_monolith.a
                |- libv8_libplatform.a
                |- libv8_libbase.a
          |- x64
             |- debug
                |- libv8_monolith.a
                |- libv8_libplatform.a
                |- libv8_libbase.a
             |- release
                |- libv8_monolith.a
                |- libv8_libplatform.a
                |- libv8_libbase.a
       |- win
           |- ia32
             |- debug
                |- v8_monolith.lib
                |- v8_libplatform.lib
                |- v8_libbase.lib
             |- release
                |- v8_monolith.lib
                |- v8_libplatform.lib
                |- v8_libbase.lib
           |- x64
             |- debug
                |- v8_monolith.lib
                |- v8_libplatform.lib
                |- v8_libbase.lib
             |- release
    |- VERSION
````

> cmake构建(CMakeLists.txt)

```
# 指出v8的 include目录
include_directories(${PROJECT_SOURCE_DIR}/v8/include)
# 判断操作系统
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")

    if (NOT(CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
        message(FATAL_ERROR "linux system only support GNU or Clang")
    endif ()

    #设置宏LINUX
    add_definitions(-DLINUX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -pthread -Wl,--no-as-needed -ldl")

    # 判断系统的是32位还是64位
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        # 64位环境下使用指针压缩
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  -DV8_COMPRESS_POINTERS")
        message(STATUS "build linux 64 ${CMAKE_BUILD_TYPE} mode")
        # 判断构建类型
        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            target_link_libraries(${PROJECT_NAME}
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/x64/debug/libv8_monolith.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/x64/debug/libv8_libbase.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/x64/debug/libv8_libplatform.a)
        else()
            target_link_libraries(${PROJECT_NAME}
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/x64/release/libv8_monolith.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/x64/release/libv8_libbase.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/x64/release/libv8_libplatform.a)
        endif()

    else()
        message(STATUS "build linux 32 ${CMAKE_BUILD_TYPE} mode")
        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            target_link_libraries(${PROJECT_NAME}
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/ia32/debug/libv8_monolith.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/ia32/debug/libv8_libbase.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/ia32/debug/libv8_libplatform.a)
        else()
            target_link_libraries(${PROJECT_NAME}
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/ia32/release/libv8_monolith.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/ia32/release/libv8_libbase.a
                    ${PROJECT_SOURCE_DIR}/v8/lib/linux/ia32/release/libv8_libplatform.a)
        endif()
    endif()

elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")

    if (NOT(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC"))
        message(SEND_ERROR "windows system only support MSVC")
    endif ()

    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        foreach(var
                CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
                CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
                CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
                CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
                )
            if(${var} MATCHES "/MDd")
                # 正则表达式替换/MD为/MT
                string(REGEX REPLACE "/MDd" "/MTd" ${var} "${${var}}")
            endif()
        endforeach()
    else()
        # 将所有默认的C,CXX编译选项中的/MD替换成/MT.
        foreach(var
                CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
                CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
                CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
                CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
                )
            if(${var} MATCHES "/MD")
                # 正则表达式替换/MD为/MT
                string(REGEX REPLACE "/MD" "/MT" ${var} "${${var}}")
            endif()
        endforeach()
    endif()


    # 设置宏 WIN
    add_definitions(-DWIN)
    # windows 链接库
    set(WINDOW_LINK_LIB winmm.lib dbghelp.lib)

    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        message(STATUS "build windows 64 ${CMAKE_BUILD_TYPE} mode")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DV8_COMPRESS_POINTERS")
        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            target_link_libraries(${PROJECT_NAME}
                    ${WINDOW_LINK_LIB}
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/x64/debug/v8_monolith.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/x64/debug/v8_libbase.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/x64/debug/v8_libplatform.lib)
        else()
            target_link_libraries(${PROJECT_NAME}
                    ${WINDOW_LINK_LIB}
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/x64/release/v8_monolith.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/x64/release/v8_libbase.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/x64/release/v8_libplatform.lib)
        endif()
    else()
        message(STATUS "build windows 32 ${CMAKE_BUILD_TYPE} mode")

        if(CMAKE_BUILD_TYPE STREQUAL "Debug")
            target_link_libraries(${PROJECT_NAME}
                    ${WINDOW_LINK_LIB}
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/ia32/debug/v8_monolith.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/ia32/debug/v8_libbase.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/ia32/debug/v8_libplatform.lib)
        else()
            target_link_libraries(${PROJECT_NAME}
                    ${WINDOW_LINK_LIB}
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/ia32/release/v8_monolith.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/ia32/release/v8_libbase.lib
                    ${PROJECT_SOURCE_DIR}/v8/lib/win/ia32/release/v8_libplatform.lib)
        endif()
    endif()
else()
    message(FATAL_ERROR "only support x86 linux and windows system")
endif()
```

## 注意事项

> linux下注意事项
+ 64位下需要启动指针压缩，需要添加编译宏 g++ 上添加 -DV8_COMPRESS_POINTERS
+ 32/64位下需要链接多线程库。g++上添加 -pthread

> windows下注意事项
+ 64位下需要启动指针压缩，V8_COMPRESS_POINTERS
+ 32/64位下需要链接库文件 winmm.lib dbghelp.lib
+ 多线程环境，debug模式下使用 MTd,release使用 MT

## 学习/赞助
v8引擎的学习文档可以访问我的博客。点击下面跳转。<br>
[v8学习文档](https://github.com/dengweichi/v8-learn)

如果需要其他系统或者编译器的静态库，可有偿找我。或者觉得有用欢迎打赏。

![RUNOOB 图标](./resource/wechat.jpg)
![RUNOOB 图标](./resource/pay.jpg)

## 构建指南

[windows构建](./doc/win-build.md)<br>
[linux构建](./doc/linux-build.md)

## 静态库下载

### v9.4 -- 更新于 2021年7月23日。
链接：https://pan.baidu.com/s/1H1UsDieawqSg2h_R3DSCaA 提取码：0000


### v11.04 -- 更新于 2023年4月15日。
链接: https://pan.baidu.com/s/14OwiR4zGE0qHqQv3gbxJ4g 提取码: 2k3w
