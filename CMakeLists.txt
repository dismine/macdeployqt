cmake_minimum_required(VERSION 3.19)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(macdeployqt)

find_package(Qt6 REQUIRED COMPONENTS Core)

set(MACDEPLOYQT_RPATH "@executable_path/../lib" CACHE STRING "RPATH in the resulting macdeployqt binary.")

# Pass -DCODE_SIGN_IDENTITY="Developer ID Application: YOUR NAME \(TEAM_ID\)" to define identity
set(CODE_SIGN_IDENTITY_DEFAULT "-" CACHE STRING "Code signing identity")

add_executable(macdeployqt
    src/main.cpp
    src/shared.cpp
    src/shared.h
    )

target_compile_definitions(macdeployqt
    PRIVATE
    QT_NO_FOREACH
    QT_USE_NODISCARD_FILE_OPEN
    )

target_link_libraries(macdeployqt
    PRIVATE
    Qt6::Core
    "-framework CoreFoundation"
    )

set_target_properties(macdeployqt PROPERTIES
    INSTALL_RPATH "${MACDEPLOYQT_RPATH}"
    )

# If CODE_SIGN_IDENTITY is not defined in the command line, use the default value
if(NOT DEFINED CODE_SIGN_IDENTITY)
    set(CODE_SIGN_IDENTITY ${CODE_SIGN_IDENTITY_DEFAULT})
endif()

# Sign the executable
if (NOT ${CODE_SIGN_IDENTITY} STREQUAL "-")
    # Certificate-based signing
    add_custom_command(TARGET macdeployqt POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E echo "Signing macdeployqt"
        COMMAND codesign -f -o runtime --timestamp -s ${CODE_SIGN_IDENTITY} $<TARGET_FILE:macdeployqt>
    )
else()
    # Ad-hoc signing
    add_custom_command(TARGET macdeployqt POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E echo "Signing \\\(ad-hoc\\\) macdeployqt"
        COMMAND codesign --force --sign ${CODE_SIGN_IDENTITY} $<TARGET_FILE:macdeployqt>
    )
endif()

install(TARGETS macdeployqt
    RUNTIME DESTINATION .
    )
