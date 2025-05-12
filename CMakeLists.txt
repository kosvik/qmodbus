# CMakeLists.txt for qmodbus (Qt6)

cmake_minimum_required(VERSION 3.16)

project(qmodbus VERSION 0.1.0 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- Find Qt6 Packages ---
# Find the required Qt6 components.
# Qt6::Widgets automatically pulls in Qt6::Gui and Qt6::Core.
find_package(Qt6 REQUIRED COMPONENTS Widgets)
qt_standard_project_setup()
# --- Automatic Qt Features ---
# Enable automatic handling of MOC, UIC, and RCC
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

# --- Project Sources ---
set(PROJECT_SOURCES
    src/main.cpp
    src/mainwindow.cpp
    src/BatchProcessor.cpp
    src/asciisettingswidget.cpp
    src/rtusettingswidget.cpp
    src/serialsettingswidget.cpp
    src/tcpipsettingswidget.cpp
    src/ipaddressctrl.cpp
    src/iplineedit.cpp

    # 3rdparty/qextserialport
    3rdparty/qextserialport/qextserialport.cpp

    # 3rdparty/libmodbus
    3rdparty/libmodbus/src/modbus.c
    3rdparty/libmodbus/src/modbus-data.c
    3rdparty/libmodbus/src/modbus-rtu.c
    3rdparty/libmodbus/src/modbus-tcp.c
    3rdparty/libmodbus/src/modbus-ascii.c
)

# --- Project Headers ---
# Listing headers is good practice, especially for IDEs,
# though not always strictly required for compilation with automoc.
set(PROJECT_HEADERS
    src/mainwindow.h
    src/BatchProcessor.h
    src/serialsettingswidget.h
    src/imodbus.h
    src/tcpipsettingswidget.h
    src/ipaddressctrl.h
    src/iplineedit.h
    src/asciisettingswidget.h
    src/rtusettingswidget.h

    # 3rdparty/qextserialport
    3rdparty/qextserialport/qextserialport.h
    3rdparty/qextserialport/qextserialenumerator.h

    # 3rdparty/libmodbus
    3rdparty/libmodbus/src/modbus.h
    3rdparty/libmodbus/src/modbus-rtu.h
    3rdparty/libmodbus/src/modbus-tcp.h
    3rdparty/libmodbus/src/modbus-ascii.h
    3rdparty/libmodbus/src/modbus-private.h      # Note: Private header usage
    3rdparty/libmodbus/src/modbus-rtu-private.h # Note: Private header usage
    3rdparty/libmodbus/src/modbus-tcp-private.h # Note: Private header usage
    3rdparty/libmodbus/src/modbus-ascii-private.h # Note: Private header usage
    3rdparty/libmodbus/config.h                # Ensure this exists or is generated
    3rdparty/libmodbus/src/modbus-version.h    # Ensure this exists or is generated
)

# --- UI Forms ---
set(PROJECT_FORMS
    forms/mainwindow.ui
    forms/about.ui
    forms/BatchProcessor.ui
    forms/serialsettingswidget.ui
    forms/tcpipsettingswidget.ui
    forms/ipaddressctrl.ui
)

# --- Resources ---/forms
set(PROJECT_RESOURCES
    data/qmodbus.qrc
)

# --- Platform Specifics ---
if(WIN32)
    list(APPEND PROJECT_SOURCES
        3rdparty/qextserialport/win_qextserialport.cpp
        3rdparty/qextserialport/qextserialenumerator_win.cpp
        3rdparty/qextserialport/qwineventnotifier_p.h # Note: Private header usage
    )
    set(PLATFORM_DEFINES
        _TTY_WIN_
        WINVER=0x0501
    )
    set(PLATFORM_LIBS setupapi ws2_32) # Windows system libraries
    set(PROJECT_RC_FILE qmodbus.rc)
else() # Assuming Unix-like if not Windows
    list(APPEND PROJECT_SOURCES
        3rdparty/qextserialport/posix_qextserialport.cpp
        3rdparty/qextserialport/qextserialenumerator_unix.cpp
    )
    set(PLATFORM_DEFINES _TTY_POSIX_)
    set(PLATFORM_LIBS "") # No extra libs specified for Unix in .pro
    set(PROJECT_RC_FILE "")
endif()

# --- Executable Target ---
add_executable(${PROJECT_NAME} WIN32 MACOSX_BUNDLE
    ${PROJECT_SOURCES}
    ${PROJECT_HEADERS} # Optional but good practice
    ${PROJECT_RESOURCES}
)

qt_add_ui(${PROJECT_NAME} INCLUDE_PREFIX "forms" SOURCES ${PROJECT_FORMS})

if(WIN32 AND PROJECT_RC_FILE)
    target_sources(${PROJECT_NAME} PRIVATE ${PROJECT_RC_FILE})
endif()

# --- Include Directories ---
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libmodbus/src
    ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/qextserialport
    # Add other necessary include paths from libmodbus if required by config.h etc.
    ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libmodbus
)

# --- Compile Definitions ---
target_compile_definitions(${PROJECT_NAME} PRIVATE ${PLATFORM_DEFINES})
# Add QT_DEPRECATED_WARNINGS if needed for migration,
# or define QT_NO_DEPRECATED_WARNINGS if you've handled deprecations
target_compile_definitions(${PROJECT_NAME} PRIVATE QT_DEPRECATED_WARNINGS)

# --- Link Libraries ---
target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt6::Widgets
    ${PLATFORM_LIBS}
)

# --- Deployment ---
# The include(deployment.pri) needs manual translation.
# Common tasks might involve installing the executable, resources, etc.
# Example (adjust paths as needed):
install(TARGETS ${PROJECT_NAME}
         RUNTIME DESTINATION bin
         BUNDLE DESTINATION . )


