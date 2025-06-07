# CMake 最低版本要求
cmake_minimum_required(VERSION 3.10)

# 项目名称和语言
project(ObjectSerialization CXX)

# 设置 C++ 标准为 C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- 定义输出目录变量 (可选，但推荐) ---
# CMAKE_BINARY_DIR 指的是 cmake 命令执行的目录，即 build/
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(TEST_EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/test)

# --- 第三方库：tinyxml2 ---
add_library(tinyxml2 STATIC thirdparty/tinyxml2/tinyxml2.cpp)
target_include_directories(tinyxml2 PUBLIC thirdparty/tinyxml2)

# --- 我们的核心库：serialization_lib ---
file(GLOB_RECURSE SERIALIZATION_SOURCES "src/*.cpp")
add_library(serialization_lib STATIC ${SERIALIZATION_SOURCES})
target_include_directories(serialization_lib PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty
)
target_link_libraries(serialization_lib PRIVATE tinyxml2)


# --- 主程序可执行文件 ---
add_executable(main_app src/main.cpp)
target_link_libraries(main_app PRIVATE serialization_lib)

# 【修改】设置 main_app 的输出目录为 ./build/
set_target_properties(main_app PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${EXECUTABLE_OUTPUT_PATH}
)


# --- 测试 ---
option(BUILD_TESTING "Build the tests" ON)

if(BUILD_TESTING)
    enable_testing()
    file(GLOB_RECURSE TEST_SOURCES "test/*.cpp")
    add_executable(run_tests ${TEST_SOURCES})
    target_link_libraries(run_tests PRIVATE serialization_lib)

    # 【修改】设置 run_tests 的输出目录为 ./build/test/
    set_target_properties(run_tests PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY ${TEST_EXECUTABLE_OUTPUT_PATH}
    )

    # CTest 会自动找到正确的路径，所以这里不需要改变
    add_test(NAME unit_tests COMMAND run_tests)
endif()
