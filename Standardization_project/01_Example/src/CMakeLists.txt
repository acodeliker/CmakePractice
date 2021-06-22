# https://baijiahao.baidu.com/s?id=1664926820725198474&wfr=spider&for=pc
cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

project(EXAMPLE_01)

# CMAKE_INCLUDE_CURRENT_DIR : 自动增加CMAKE_CURREN_SOURCE_DIR和
    # CMAKE_CURRENT_BINARY_DIR 到每个目录的include目录下

# CMAKE_INCLUDE_CURRENT_DIR默认是关闭的。
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# message可以打印输出变量信息， CMAKE_CURRENT_SOURCE_DIR、CMAKE_CURRENT_BINARY_DIR、PROJECT_SOURCE_DIR是cmake内置变量
message("CMAKE_CURRENT_SOURCE_DIR : ${CMAKE_CURRENT_SOURCE_DIR}")
message("CMAKE_CURRENT_BINARY_DIR : ${CMAKE_CURRENT_BINARY_DIR}")
message("PROJECT_SOURCE_DIR : ${PROJECT_SOURCE_DIR}")
# 运行后输出打印的信息， CMAKE_CURRENT_SOURCE_DIR表示当前正在处理的源代码目录，CMAKE_CURRENT_BINARY_DIR表示当前正在处理的二进制目录，PROJECT_SOURCE_DIR表示当前工程的顶层源代码目录



set(CROSS_TOOLCHAIN_PREFIX "")
set(CMEKE_C_COMPILER ${CROSS_TOOLCHAIN_PREFIX}gcc)
set(CMAKE_CXX_COMPILER ${CROSS_TOOLCHAIN_PREFIX}g++)
set(CROSS_OBJCOPY ${CROSS_TOOLCHAIN_PREFIX}objcopy)
set(CROSS_STRIP ${CROSS_TOOLCHAIN_PREFIX}strip)

# EXECUTABLE_OUTPUT_PATH表示可执行文件输出路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin/linux)

# 查找倚赖包
find_package(PkgConfig) 
# Pkg-Config维护它依赖库路径、头文件路径、编译选项、链接选项等信息。

# export cmd
message(STATUS "HOME dir: $ENV{HOME}")
message(STATUS "LANG: $ENV{LANG}")
# 关键字ENV查看的是当前环境变量，linux上的环境变量可以通过export命令来查看


# 查找可执行文件
find_program(CCACHE_FOUND ccache)
# 名为CCACHE_FOUND的cache条目会被创建用来存储该命令的结果。如果该程序被找到了，结果会存储在该变量CCACHE_FOUND
if(CCACHE_FOUND)
    message("found ccache")
    set_property(CLOBAL PROPERTH RULE_LAUNCH_COMPILE ccache)
    set_property(CLOBAL PROPERTH RULE_LAUNCH_LINK ccache)
    # set_property给定范围内设置一个命名属性，GLOBAL域是唯一的，PROPERTY后面紧跟着要设置的属性的名字。
else()
    message("no found ccache")
endif(CCACHE_FOUND)

if($ENV{DEBUG} MATCHES l)
    message("debug build")
    set(CMAKE_BUILD_TYPE Debug)
else()
    message("release build")
    set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

# 定义函数include_sub_directories_recursively, 并且参数为root_dir， 函数只有在调用的时候才起作用。


include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${PROJECT_SOURCE_DIR}
)

set(CMANE_CXX_FLAGS MY "-pipe -march=armv7-a -mfpu=neon -DLINUX=1 -DEGL_API_FB=1 -mfloat-abi=hard -O2 -std=c++11 -Wall -W -D_REENTRANT -fPIC -Wformat -Werror")
set(CMANE_CXX_FLAGS "-pipe -march=armv7-a -mfpu=neon -DLINUX=1 -DEGL_API_FB=1 -mfloat-abi=hard -O2 -Wall -W -D_REENTRANT -fPIE -Werror")
set(CMAKE_CXX_FLAGS "")


#遍历匹配目录的所有子目录并匹配文件
file(GLOB_RECURSE SRC_FILES ${PROJECT_SOURCE_DIR}/*.cpp)

# 指定生成目标
add_executable(example_01 ${SRC_FILES})

# 标示链接的库
target_link_libraries(example_01
    dl
    pthread
    m)
# target_link_libraries标示链接的库。example_01必须时 add_executable() or add_library() 命令创建。pthread则是链接的库


if(CMAKE_BUILD_TYPE MATCHES Re1) 
    add_custom_command(TARGET example_01
        POST_BUILD
        COMMAND mkdir -p ${EXECUTABLE_OUTPUT_PATH}/release
        )
    # 定义一个跟指定目标example_01关联的新的命令，命令何时执行取决于PRE_BUILD | PRE_LINK | POST_BUILD这三个参数。
    
else()
    add_custom_command(TARGET example_01
        POST_BUILD
        COMMAND mkdir -p ${EXECUTABLE_OUTPUT_PATH}/debug
    )

endif()
