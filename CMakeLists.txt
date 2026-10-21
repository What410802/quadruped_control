# 文件：CMakeLists.txt
# 作用：配置 quadruped_control 工程及其核心库和测试入口。

cmake_minimum_required(VERSION 3.16)

project(quadruped_control VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(QUADRUPED_ENABLE_MUJOCO "构建需要 MuJoCo 的仿真模块" OFF)
option(QUADRUPED_ENABLE_TORCH "构建需要固定 LibTorch 的策略模块" OFF)

if(QUADRUPED_ENABLE_MUJOCO)
    # Pixi environment provides MuJoCo 3.12 with a CMake config package.
    find_package(mujoco CONFIG REQUIRED)
endif()

add_subdirectory(core)
add_subdirectory(adapters/ipc)
add_subdirectory(motion)
add_subdirectory(config_loader)
add_subdirectory(backends/replay)

if(QUADRUPED_ENABLE_MUJOCO)
    add_subdirectory(backends/mujoco)
endif()

if(QUADRUPED_ENABLE_TORCH)
    # Use the Pixi-provided LibTorch; no repository-local .deps tree.
    find_package(Torch CONFIG REQUIRED)
    add_subdirectory(policy/torch)
endif()

option(QUADRUPED_BUILD_APPS "Build GUI/daemon apps (requires MuJoCo Simulate sources)" ON)
if(QUADRUPED_BUILD_APPS)
    add_subdirectory(apps)
endif()

include(CTest)
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()
