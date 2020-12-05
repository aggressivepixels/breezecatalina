cmake_minimum_required(VERSION 3.0)
project(breezecatalina)
set(PROJECT_VERSION "5.20.80")
set(PROJECT_VERSION_MAJOR 5)

set(KF5_MIN_VERSION "5.74")
set(QT_MIN_VERSION "5.15.0")

include(GenerateExportHeader)
include(WriteBasicConfigVersionFile)
include(FeatureSummary)

find_package(ECM ${KF5_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake)

include(ECMInstallIcons)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDEClangFormat)

find_package(KDecoration2 REQUIRED)

add_definitions(-DTRANSLATION_DOMAIN="breeze_kwin_deco")

find_package(KF5 REQUIRED COMPONENTS CoreAddons GuiAddons ConfigWidgets WindowSystem I18n IconThemes)
find_package(Qt5 CONFIG REQUIRED COMPONENTS DBus)

### XCB
find_package(XCB COMPONENTS XCB)
set_package_properties(XCB PROPERTIES
  DESCRIPTION "X protocol C-language Binding"
  URL "https://xcb.freedesktop.org"
  TYPE OPTIONAL
  PURPOSE "Required to pass style properties to native Windows on X11 Platform"
)

if(UNIX AND NOT APPLE)

  set(BREEZE_HAVE_X11 ${XCB_XCB_FOUND})
  if (XCB_XCB_FOUND)
    find_package(Qt5 REQUIRED CONFIG COMPONENTS X11Extras)
  endif()

else()

  set(BREEZE_HAVE_X11 FALSE)

endif()

################# configuration #################
configure_file(config-breeze.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-breeze.h )

################# newt target #################
### plugin classes
set(breezecatalina_SRCS
    breezebutton.cpp
    breezedecoration.cpp
    breezeexceptionlist.cpp
    breezesettingsprovider.cpp
    breezesizegrip.cpp
    breezeboxshadowrenderer.cpp)

kconfig_add_kcfg_files(breezecatalina_SRCS breezesettings.kcfgc)

### config classes
### they are kept separately because they might move in a separate library in the future
set(breezecatalina_config_SRCS
    config/breezeconfigwidget.cpp
    config/breezedetectwidget.cpp
    config/breezeexceptiondialog.cpp
    config/breezeexceptionlistwidget.cpp
    config/breezeexceptionmodel.cpp
    config/breezeitemmodel.cpp
)

set(breezecatalina_config_PART_FORMS
   config/ui/breezeconfigurationui.ui
   config/ui/breezedetectwidget.ui
   config/ui/breezeexceptiondialog.ui
   config/ui/breezeexceptionlistwidget.ui
)

ki18n_wrap_ui(breezecatalina_config_PART_FORMS_HEADERS ${breezecatalina_config_PART_FORMS})

### build library
add_library(breezecatalina MODULE
    ${breezecatalina_SRCS}
    ${breezecatalina_config_SRCS}
    ${breezecatalina_config_PART_FORMS_HEADERS})

target_link_libraries(breezecatalina
    PUBLIC
        Qt5::Core
        Qt5::Gui
        Qt5::DBus
    PRIVATE
        KDecoration2::KDecoration
        KF5::ConfigCore
        KF5::CoreAddons
        KF5::ConfigWidgets
        KF5::GuiAddons
        KF5::I18n
        KF5::IconThemes
        KF5::WindowSystem)

if(BREEZE_HAVE_X11)
  target_link_libraries(breezecatalina
    PUBLIC
      Qt5::X11Extras
      XCB::XCB)
endif()

install(TARGETS breezecatalina DESTINATION ${PLUGIN_INSTALL_DIR}/org.kde.kdecoration2)
install(FILES config/breezecatalinaconfig.desktop DESTINATION ${SERVICES_INSTALL_DIR})
