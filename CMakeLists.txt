cmake_minimum_required(VERSION 3.16)
project("ledit")
set(VERSION "0.0.4")
set(CMAKE_CXX_STANDARD 17)

# Hack found here (https://stackoverflow.com/a/32259072)
if(UNIX AND NOT APPLE)
    set(LINUX TRUE)
endif()

if (WIN32 OR LINUX OR APPLE)
    # Use embeded glfw freetype2 and glad
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

    add_subdirectory(third-party/glfw)
    add_subdirectory(third-party/freetype2)
    include_directories(PUBLIC
        third-party/glad
        third-party/json
    )
elseif (NINTENDO_SWITCH)
    # Use portlib third-party but json
    set(NRO_AUTHOR "liz3 / romain337 (port)")
    set(NRO_ROMFS ${PROJECT_SOURCE_DIR}/assets/romfs)
    set(NRO_ICON ${PROJECT_SOURCE_DIR}/assets/icon.jpg)

    find_package(glfw3 REQUIRED)
    find_package(Freetype REQUIRED)
    find_package(OpenGL REQUIRED)
    find_package(BZip2 REQUIRED)
    find_package(ZLIB REQUIRED)
    find_package(PNG REQUIRED)
    include_directories(PUBLIC
        ${FREETYPE_INCLUDE_DIRS}
        third-party/json
    )
endif()

# Project sources files
list(APPEND SRCS
    src/la.cc
    src/main.cc
)

if (NOT NINTENDO_SWITCH)
    # Nintendo Switch use it's own version from devkitpro
    list(APPEND SRCS
        third-party/glad/glad/glad.c
    )
endif()

add_executable(ledit ${SRCS})

if (NINTENDO_SWITCH)
    # Nintendo Switch generate app information for hbl
    nx_generate_nacp(${PROJECT_BINARY_DIR}/${PROJECT_NAME}.nacp
        NAME ${PROJECT_NAME}
        AUTHOR ${NRO_AUTHOR}
        VERSION ${VERSION}
    )

    # Nintendo Switch generate executable nro from elf
    nx_create_nro(ledit
        NACP ${PROJECT_NAME}.nacp
        ICON ${NRO_ICON}
        #ROMFS ${NRO_ROMFS}
    )
endif()

list(APPEND LIBS)

if (LINUX)
    list(APPEND LIBS
        glfw
        freetype
        fontconfig
        dl
    )
elseif (WIN32 OR APPLE)
    list(APPEND LIBS PUBLIC
        glfw
        freetype
    )
elseif (NINTENDO_SWITCH)
    list(APPEND LIBS
        glfw
        glad
        ${OPENGL_LIBRARIES}
        ${FREETYPE_LIBRARIES}
        ${BZIP2_LIBRARIES}
        ${ZLIB_LIBRARIES}
        ${PNG_LIBRARIES}
    )
endif()

target_link_libraries(ledit ${LIBS})

if(APPLE)
    #set(CMAKE_CXX_FLAGS_RELEASE "-o3")
endif()
