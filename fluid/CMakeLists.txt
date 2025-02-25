#
# CMakeLists.txt to build fluid for the FLTK project using CMake (www.cmake.org)
#
# Copyright 1998-2021 by Bill Spitzak and others.
#
# This library is free software. Distribution and use rights are outlined in
# the file "COPYING" which should have been included with this file.  If this
# file is missing or damaged, see the license at:
#
#     https://www.fltk.org/COPYING.php
#
# Please see the following page on how to report bugs and issues:
#
#     https://www.fltk.org/bugs.php
#

if (NOT ANDROID)

set (CPPFILES
  CodeEditor.cxx
  StyleParse.cxx
  Fl_Function_Type.cxx
  Fl_Group_Type.cxx
  Fl_Menu_Type.cxx
  Fl_Type.cxx
  Fl_Widget_Type.cxx
  Fl_Window_Type.cxx
  Fluid_Image.cxx
  about_panel.cxx
  align_widget.cxx
  alignment_panel.cxx
  code.cxx
  factory.cxx
  file.cxx
  fluid.cxx
  function_panel.cxx
  template_panel.cxx
  undo.cxx
  widget_panel.cxx
)

# ExternalCodeEditor: platform specific files

if (WIN32)
  list (APPEND CPPFILES ExternalCodeEditor_WIN32.cxx)
else ()
  list (APPEND CPPFILES ExternalCodeEditor_UNIX.cxx)
endif (WIN32)

if (APPLE AND (NOT OPTION_APPLE_X11) AND (NOT OPTION_APPLE_SDL))
  set (ICON_NAME fluid.icns)
  set (ICON_PATH "${PROJECT_SOURCE_DIR}/fluid/fluid.app/Contents/Resources/${ICON_NAME}")
  add_executable (fluid MACOSX_BUNDLE ${CPPFILES} ${ICON_PATH})

  # create macOS bundle wrapper script

  set (WRAPPER "${EXECUTABLE_OUTPUT_PATH}/${CMAKE_CFG_INTDIR}/fluid")
  add_custom_command (
    TARGET fluid POST_BUILD
    COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/../CMake/macOS-bundle-wrapper.in ${WRAPPER}
    COMMAND chmod u+x,g+x,o+x ${WRAPPER}
    BYPRODUCTS ${WRAPPER}
    # COMMENT "Creating macOS bundle wrapper script ${WRAPPER}"
    VERBATIM
  )
  unset (WRAPPER)

else ()
  add_executable (fluid WIN32 ${CPPFILES})
endif (APPLE AND (NOT OPTION_APPLE_X11) AND (NOT OPTION_APPLE_SDL))

target_link_libraries (fluid fltk fltk_images fltk_forms)
if (FLTK_HAVE_CAIRO)
  target_link_directories (fluid PUBLIC ${PKG_CAIRO_LIBRARY_DIRS})
endif (FLTK_HAVE_CAIRO)

# install fluid

if (APPLE AND (NOT OPTION_APPLE_X11) AND (NOT OPTION_APPLE_SDL))
  # create bundle
  set_target_properties (fluid PROPERTIES MACOSX_BUNDLE_INFO_PLIST "${PROJECT_SOURCE_DIR}/fluid/fluid.app/Contents/Info.plist")
  set_target_properties (fluid PROPERTIES MACOSX_BUNDLE_ICON_FILE ${ICON_NAME})
  set_target_properties (fluid PROPERTIES RESOURCE ${ICON_PATH})
  # install
  install (TARGETS fluid DESTINATION ${FLTK_BINDIR})
else()
  install (TARGETS fluid
    EXPORT FLTK-Targets
    RUNTIME DESTINATION ${FLTK_BINDIR}
    LIBRARY DESTINATION ${FLTK_LIBDIR}
    ARCHIVE DESTINATION ${FLTK_LIBDIR}
  )
endif (APPLE AND (NOT OPTION_APPLE_X11) AND (NOT OPTION_APPLE_SDL))

# install desktop files

if (UNIX)
  install (FILES fluid.desktop
    DESTINATION ${FLTK_DATADIR}/applications
  )
  # Install mime-type file (x-fluid.desktop method is deprecated)
  install (FILES fluid.xml
    DESTINATION ${FLTK_DATADIR}/mime/packages
  )

  # Install desktop icons
  foreach (icon 32 48 64 128)
    install (FILES icons/fluid-${icon}.png
      DESTINATION ${FLTK_DATADIR}/icons/hicolor/${icon}x${icon}/apps
      RENAME fluid.png
    )
  endforeach()
endif (UNIX)

endif (NOT ANDROID)
