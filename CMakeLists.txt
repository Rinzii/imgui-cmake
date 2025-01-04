# The library target, e.g.
add_library(imgui)

# Tweak the install interface to point to top-level "include"
target_include_directories(imgui
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
)

# The usual list of ImGui source files:
set(IMGUI_SOURCES
    imgui.cpp
    imgui.h
    imgui_demo.cpp
    imgui_draw.cpp
    imgui_tables.cpp
    imgui_widgets.cpp
    imstb_rectpack.h
    imstb_textedit.h
    imstb_truetype.h
)

target_sources(imgui PRIVATE ${IMGUI_SOURCES})

# Only if you want to install ImGui
if(IMGUI_INSTALL)
    install(TARGETS imgui
        EXPORT imguiTargets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include  # <= points to "include"
    )
    install(FILES
        imgui.h
        imconfig.h
        imgui_internal.h
        imstb_rectpack.h
        imstb_textedit.h
        imstb_truetype.h
        DESTINATION include          # <= root of "include" not "include/imgui"
    )

    install(EXPORT imguiTargets
        FILE imguiTargets.cmake
        NAMESPACE ImGui::
        DESTINATION lib/cmake/imgui
    )
endif()
