cmake_minimum_required(VERSION 3.30)

project(VulkanLearning LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(VulkanLearning
        main.cpp
)

find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)
find_package(tinyobjloader CONFIG REQUIRED)

find_path(STB_INCLUDE_DIRS "stb_image.h")

target_include_directories(VulkanLearning PRIVATE
        ${STB_INCLUDE_DIRS}
)

target_link_libraries(VulkanLearning PRIVATE
        Vulkan::Vulkan
        glfw
        glm::glm
        tinyobjloader::tinyobjloader
)

find_program(SLANGC_EXECUTABLE slangc REQUIRED)

function(add_slang_shader_target TARGET)
    cmake_parse_arguments(SHADER "" "" "SOURCES" ${ARGN})

    set(SHADERS_OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/shaders)
    set(SHADER_OUTPUT_FILE ${SHADERS_OUTPUT_DIR}/shader.spv)

    add_custom_command(
            OUTPUT ${SHADER_OUTPUT_FILE}
            COMMAND ${CMAKE_COMMAND} -E make_directory ${SHADERS_OUTPUT_DIR}
            COMMAND ${SLANGC_EXECUTABLE}
            ${SHADER_SOURCES}
            -target spirv
            -profile spirv_1_4
            -emit-spirv-directly
            -fvk-use-entrypoint-name
            -entry vertMain
            -entry fragMain
            -o ${SHADER_OUTPUT_FILE}
            DEPENDS ${SHADER_SOURCES}
            COMMENT "Compiling Slang shader"
            VERBATIM
    )

    add_custom_target(${TARGET} DEPENDS ${SHADER_OUTPUT_FILE})
endfunction()

set(SHADER_SLANG_SOURCES "${CMAKE_CURRENT_LIST_DIR}/shaders/shader.slang")

add_slang_shader_target(compile_slang_shaders SOURCES ${SHADER_SLANG_SOURCES})
add_dependencies(VulkanLearning compile_slang_shaders)