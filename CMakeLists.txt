cmake_minimum_required(VERSION 3.18)
project(imgui LANGUAGES CXX)

option(IMGUI_ENABLE_OPENGL2 "Enable OpenGL2 backend" OFF)
option(IMGUI_ENABLE_OPENGL3 "Enable OpenGL3 backend" OFF)
option(IMGUI_ENABLE_GLFW "Enable GLFW backend" OFF)
option(IMGUI_ENABLE_SDL2 "Enable SDL2 backend" OFF)
option(IMGUI_ENABLE_SDL3 "Enable SDL3 backend" OFF)
option(IMGUI_ENABLE_SDLRENDERER2 "Enable SDLRenderer2 backend" OFF)
option(IMGUI_ENABLE_SDLRENDERER3 "Enable SDLRenderer3 backend" OFF)
option(IMGUI_ENABLE_VULKAN "Enable Vulkan backend" OFF)
option(IMGUI_ENABLE_WIN32 "Enable Win32 backend" OFF)
option(IMGUI_ENABLE_METAL "Enable Metal backend" OFF)
option(IMGUI_ENABLE_ALLEGRO5 "Enable Allegro5 backend" OFF)
option(IMGUI_ENABLE_ANDROID "Enable Android backend" OFF)
option(IMGUI_ENABLE_DX9 "Enable DirectX9 backend" OFF)
option(IMGUI_ENABLE_DX10 "Enable DirectX10 backend" OFF)
option(IMGUI_ENABLE_DX11 "Enable DirectX11 backend" OFF)
option(IMGUI_ENABLE_DX12 "Enable DirectX12 backend" OFF)
option(IMGUI_ENABLE_WGPU "Enable WebGPU backend" OFF)
option(IMGUI_ENABLE_OSX "Enable OSX backend" OFF)
option(IMGUI_DISABLE_FIND_PACKAGE "Disable find_package and assume targets exist" OFF)
option(IMGUI_INSTALL "Enable installation of the library" OFF)
option(IMGUI_BRING_YOUR_OWN_BACKEND "Disable backend check and assume user provides their own backend" OFF)
option(IMGUI_DISABLE_SET_IMGUI_DIR "Disable setting the IMGUI_DIR variable" OFF)

add_library(${PROJECT_NAME})

#
# Make sure #include <imgui.h> works at build-time and install-time.
#
target_include_directories(${PROJECT_NAME}
    PUBLIC
        # Where to find headers during build:
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>

        # Where to look for headers once installed (root-level include/):
        $<INSTALL_INTERFACE:include>
)

#
# Core ImGui sources
#
set(IMGUI_SOURCES
    imconfig.h
    imgui.cpp
    imgui.h
    imgui_demo.cpp
    imgui_draw.cpp
    imgui_internal.h
    imgui_tables.cpp
    imgui_widgets.cpp
    imstb_rectpack.h
    imstb_textedit.h
    imstb_truetype.h
)

# (Optional) Add backends here if you enable them via IMGUI_ENABLE_ flags

# If user doesn't supply their own backend and none are enabled, throw an error
if(NOT IMGUI_BRING_YOUR_OWN_BACKEND)
    if(
        NOT IMGUI_ENABLE_OPENGL2
        AND NOT IMGUI_ENABLE_OPENGL3
        AND NOT IMGUI_ENABLE_GLFW
        AND NOT IMGUI_ENABLE_SDL2
        AND NOT IMGUI_ENABLE_SDL3
        AND NOT IMGUI_ENABLE_SDLRENDERER2
        AND NOT IMGUI_ENABLE_SDLRENDERER3
        AND NOT IMGUI_ENABLE_VULKAN
        AND NOT IMGUI_ENABLE_WIN32
        AND NOT IMGUI_ENABLE_METAL
        AND NOT IMGUI_ENABLE_ALLEGRO5
        AND NOT IMGUI_ENABLE_ANDROID
        AND NOT IMGUI_ENABLE_DX9
        AND NOT IMGUI_ENABLE_DX10
        AND NOT IMGUI_ENABLE_DX11
        AND NOT IMGUI_ENABLE_DX12
        AND NOT IMGUI_ENABLE_WGPU
        AND NOT IMGUI_ENABLE_OSX
    )
        message(FATAL_ERROR "No backend has been enabled. Please enable at least one backend, or use IMGUI_BRING_YOUR_OWN_BACKEND.")
    endif()
endif()

#
# Optionally store the top-level ImGui path in IMGUI_DIR
#
if(NOT IMGUI_DISABLE_SET_IMGUI_DIR)
    set(IMGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR} CACHE PATH "ImGui top-level directory")
endif()

target_sources(${PROJECT_NAME} PRIVATE ${IMGUI_SOURCES})

#
# Compiler options
#
if(CMAKE_CXX_COMPILER_ID STREQUAL Clang OR CMAKE_CXX_COMPILER_ID STREQUAL GNU)
    target_compile_options(${PROJECT_NAME} PRIVATE
        -Wno-conversion
    )
endif()

#
# Install logic: place library + headers so #include <imgui.h> works
#
if(IMGUI_INSTALL)
    install(TARGETS ${PROJECT_NAME}
        EXPORT       imguiTargets
        LIBRARY      DESTINATION lib
        ARCHIVE      DESTINATION lib
        RUNTIME      DESTINATION bin
        INCLUDES     DESTINATION include
    )

    install(FILES
        imgui.h
        imconfig.h
        imgui_internal.h
        imstb_rectpack.h
        imstb_textedit.h
        imstb_truetype.h
        DESTINATION include  # <-- Puts them in the root-level "include/" folder
    )

    install(EXPORT imguiTargets
        FILE      imguiTargets.cmake
        NAMESPACE ImGui::
        DESTINATION lib/cmake/imgui
    )
endif()
