cmake_minimum_required(VERSION 3.18)
project(imgui)

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

# Define the library
add_library(${PROJECT_NAME})

# Point to the `imgui` submodule as the source root
set(IMGUI_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/imgui)

# Add ImGui core sources
set(IMGUI_SOURCES
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

# If backends are enabled, append their sources
if(IMGUI_ENABLE_OPENGL2)
    list(APPEND IMGUI_SOURCES
        ${IMGUI_ROOT}/backends/imgui_impl_opengl2.cpp
        ${IMGUI_ROOT}/backends/imgui_impl_opengl2.h
    )
endif()

if(IMGUI_ENABLE_OPENGL3)
    list(APPEND IMGUI_SOURCES
        ${IMGUI_ROOT}/backends/imgui_impl_opengl3.cpp
        ${IMGUI_ROOT}/backends/imgui_impl_opengl3.h
        ${IMGUI_ROOT}/backends/imgui_impl_opengl3_loader.h
    )
endif()

if(IMGUI_ENABLE_GLFW)
    list(APPEND IMGUI_SOURCES
        ${IMGUI_ROOT}/backends/imgui_impl_glfw.cpp
        ${IMGUI_ROOT}/backends/imgui_impl_glfw.h
    )
    if(NOT IMGUI_DISABLE_FIND_PACKAGE)
        find_package(glfw3 REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PRIVATE glfw)
endif()

if(IMGUI_ENABLE_SDL2)
    list(APPEND IMGUI_SOURCES
        ${IMGUI_ROOT}/backends/imgui_impl_sdl2.cpp
        ${IMGUI_ROOT}/backends/imgui_impl_sdl2.h
    )
    if(NOT IMGUI_DISABLE_FIND_PACKAGE)
        find_package(SDL2 REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PRIVATE SDL2::SDL2)
endif()

if(IMGUI_ENABLE_SDL3)
    list(APPEND IMGUI_SOURCES
        ${IMGUI_ROOT}/backends/imgui_impl_sdl3.cpp
        ${IMGUI_ROOT}/backends/imgui_impl_sdl3.h
    )
    if(NOT IMGUI_DISABLE_FIND_PACKAGE)
        find_package(SDL3 REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PRIVATE SDL3::SDL3)
endif()

# Ensure build-time and install-time includes work
target_include_directories(${PROJECT_NAME}
    INTERFACE
        $<BUILD_INTERFACE:${IMGUI_ROOT}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/imgui>
)

# Add the sources to the library
target_sources(${PROJECT_NAME} PRIVATE ${IMGUI_SOURCES})

# Compiler warnings
if(CMAKE_CXX_COMPILER_ID STREQUAL Clang OR CMAKE_CXX_COMPILER_ID STREQUAL GNU)
    target_compile_options(${PROJECT_NAME} PRIVATE
        -Wno-conversion
    )
endif()

# Installation logic
if(IMGUI_INSTALL)
    include(GNUInstallDirs)

    # Install the library
    install(TARGETS ${PROJECT_NAME}
        EXPORT imguiTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imgui
    )

    # Install core header files
    install(FILES
        ${IMGUI_ROOT}/imgui.h
        ${IMGUI_ROOT}/imconfig.h
        ${IMGUI_ROOT}/imgui_internal.h
        ${IMGUI_ROOT}/imstb_rectpack.h
        ${IMGUI_ROOT}/imstb_textedit.h
        ${IMGUI_ROOT}/imstb_truetype.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imgui
    )

    # Install backends (optional, based on user needs)
    install(DIRECTORY ${IMGUI_ROOT}/backends
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imgui
        FILES_MATCHING PATTERN "*.h"
    )

    # Install CMake configuration files for find_package
    install(EXPORT imguiTargets
        FILE imguiTargets.cmake
        NAMESPACE ImGui::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/imgui
    )

    include(CMakePackageConfigHelpers)
    write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        COMPATIBILITY SameMajorVersion
    )

    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/imguiConfig.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        @ONLY
    )

    install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/imgui
    )
endif()
