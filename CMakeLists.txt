cmake_minimum_required(VERSION 3.10)
project(mlx90640 LANGUAGES C CXX)

set(I2C_MODE LINUX CACHE STRING "Set the I2C mode, options are 'LINUX' or 'RPI'")

if (${I2C_MODE} STREQUAL "RPI")
    find_library(BCM2835_LIB bcm2835)
    if(NOT BCM2835_LIB)
        message(FATAL_ERROR "You can find library here: http://www.airspayce.com/mikem/bcm2835/")
    endif()
endif()

find_package(SDL2 REQUIRED)
include_directories(${SDL2_INCLUDE_DIRS})

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
include_directories(headers)

# Add the static library target
add_library(${PROJECT_NAME}
        STATIC
        functions/MLX90640_API.cpp
        functions/MLX90640_${I2C_MODE}_I2C_Driver.cpp
        headers/MLX90640_API.h
        headers/MLX90640_I2C_Driver.h
)

add_library(${PROJECT_NAME}_extra
    STATIC
    examples/src/lib/fb.c
    examples/src/lib/fb.h
    examples/src/lib/interpolate.c
    examples/src/lib/interpolate.h
)

# Add the shared library target
add_library(${PROJECT_NAME}_shared
    SHARED
    functions/MLX90640_API.cpp
    ${I2C_DRIVER_SRC}
    headers/MLX90640_API.h
    headers/MLX90640_I2C_Driver.h
)
set_target_properties(${PROJECT_NAME}_shared PROPERTIES OUTPUT_NAME ${PROJECT_NAME})

add_library(${PROJECT_NAME}_extra_shared
        SHARED
        examples/src/lib/fb.c
        examples/src/lib/fb.h
        examples/src/lib/interpolate.c
        examples/src/lib/interpolate.h
)
set_target_properties(${PROJECT_NAME}_extra_shared PROPERTIES OUTPUT_NAME ${PROJECT_NAME}_extra)

add_executable(test examples/src/test.cpp)
target_link_libraries(test ${PROJECT_NAME})

add_executable(fbuf examples/src/fbuf.cpp)
target_link_libraries(fbuf PRIVATE ${PROJECT_NAME})
target_link_libraries(fbuf PRIVATE ${PROJECT_NAME}_extra)

add_executable(hotspot examples/src/hotspot.cpp)
target_link_libraries(hotspot PRIVATE ${PROJECT_NAME})
target_link_libraries(hotspot PRIVATE ${PROJECT_NAME}_extra)

add_executable(interp examples/src/interp.cpp)
target_link_libraries(interp PRIVATE ${PROJECT_NAME})
target_link_libraries(interp PRIVATE ${PROJECT_NAME}_extra)

add_executable(rawrgb examples/src/rawrgb.cpp)
target_link_libraries(rawrgb PRIVATE ${PROJECT_NAME})

add_executable(sdlscale examples/src/sdlscale.cpp)
target_link_libraries(sdlscale PRIVATE ${PROJECT_NAME})
target_link_libraries(sdlscale PRIVATE ${SDL2_LIBRARIES})

add_executable(step examples/src/step.cpp)
target_link_libraries(step PRIVATE ${PROJECT_NAME})

add_executable(video examples/src/video.cpp)
target_link_libraries(video PRIVATE ${PROJECT_NAME})
target_link_libraries(video PRIVATE ${PROJECT_NAME}_extra)
target_link_libraries(video PRIVATE -lavcodec -lavutil -lavformat)
