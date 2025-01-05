cmake_minimum_required(VERSION 3.18)
project(imgui VERSION 1.91.6)

# Backend options
set(BACKENDS
    OPENGL2
    OPENGL3
    GLFW
    SDL2
    SDL3
    SDLRENDERER2
    SDLRENDERER3
    VULKAN
    WIN32
    METAL
    ALLEGRO5
    ANDROID
    DX9
    DX10
    DX11
    DX12
    WGPU
    OSX
)

foreach(BACKEND IN LISTS BACKENDS)
    string(TOUPPER ${BACKEND} BACKEND_UPPER)
    option(IMGUI_ENABLE_${BACKEND_UPPER} "Enable ${BACKEND} backend" OFF)
endforeach()

# Other options
option(IMGUI_DISABLE_FIND_PACKAGE "Disable find_package and assume targets exist" OFF)
option(IMGUI_BRING_YOUR_OWN_BACKEND "Disable backend check and assume user provides their own backend" OFF)
option(IMGUI_DISABLE_SET_IMGUI_DIR "Disable setting the IMGUI_DIR variable" OFF)

# Define the library
add_library(${PROJECT_NAME})
target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC ./imgui)

# ImGui source files
set(IMGUI_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
if(NOT IMGUI_DISABLE_SET_IMGUI_DIR)
    set(IMGUI_DIR ${IMGUI_ROOT} CACHE PATH "ImGui top-level directory")
endif()

target_sources(${PROJECT_NAME} PRIVATE
    ${IMGUI_ROOT}/imconfig.h
    ${IMGUI_ROOT}/imgui.cpp
    ${IMGUI_ROOT}/imgui.h
    ${IMGUI_ROOT}/imgui_demo.cpp
    ${IMGUI_ROOT}/imgui_draw.cpp
    ${IMGUI_ROOT}/imgui_internal.h
    ${IMGUI_ROOT}/imgui_tables.cpp
    ${IMGUI_ROOT}/imgui_widgets.cpp
    ${IMGUI_ROOT}/imstb_rectpack.h
    ${IMGUI_ROOT}/imstb_textedit.h
    ${IMGUI_ROOT}/imstb_truetype.h
)

# Backend handling
foreach(BACKEND IN LISTS BACKENDS)
    string(TOUPPER ${BACKEND} BACKEND_UPPER)
    if(IMGUI_ENABLE_${BACKEND_UPPER})
        string(TOLOWER ${BACKEND} BACKEND_LOWER)
        list(APPEND BACKEND_SOURCES ${IMGUI_ROOT}/backends/imgui_impl_${BACKEND_LOWER}.cpp
                                    ${IMGUI_ROOT}/backends/imgui_impl_${BACKEND_LOWER}.h)
        if(IMGUI_ENABLE_GLFW AND NOT IMGUI_DISABLE_FIND_PACKAGE)
            find_package(glfw3 REQUIRED)
            target_link_libraries(${PROJECT_NAME} PRIVATE glfw)
        endif()
        if(IMGUI_ENABLE_SDL2 AND NOT IMGUI_DISABLE_FIND_PACKAGE)
            find_package(SDL2 REQUIRED)
            target_link_libraries(${PROJECT_NAME} PRIVATE SDL2::SDL2)
        endif()
        if(IMGUI_ENABLE_SDL3 AND NOT IMGUI_DISABLE_FIND_PACKAGE)
            find_package(SDL3 REQUIRED)
            target_link_libraries(${PROJECT_NAME} PRIVATE SDL3::SDL3)
        endif()
    endif()
endforeach()

target_sources(${PROJECT_NAME} PRIVATE ${BACKEND_SOURCES})

# Compiler warnings
if(CMAKE_CXX_COMPILER_ID STREQUAL Clang OR CMAKE_CXX_COMPILER_ID STREQUAL GNU)
    target_compile_options(${PROJECT_NAME} PRIVATE -Wno-conversion)
endif()
