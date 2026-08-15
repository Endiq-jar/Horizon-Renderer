cmake_minimum_required(VERSION 3.10)

project(NW
    VERSION 0.1.0
    LANGUAGES C CXX
)

# ============================================================
# Build configuration
# ============================================================

include(CheckIPOSupported)

check_ipo_supported(
    RESULT LTOSupported
    OUTPUT LTOError
)

if(LTOSupported)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    add_compile_options(
        -g0
        -O3
        -ffunction-sections
        -fdata-sections
        -fomit-frame-pointer
        -fmerge-all-constants
    )

    add_compile_definitions(
        NDEBUG
        # Skip printf()/write_log() formatting on every SHUT_LOGD/LOGD/LOGE
        # call site (texture uploads, FBO rebinds, buffer binds, etc. all
        # hit these paths); only startup banner logging goes through
        # LogPrintf_NoPrefix, which stays unaffected.
        GL4ES_SILENCE_MESSAGES
    )

    add_link_options(
        -Wl,--gc-sections
    )

    # 32-bit ARM devices (older phones) don't get NEON/VFP codegen unless
    # explicitly requested; arm64-v8a already has NEON mandatorily so this
    # is a no-op there.
    if(ANDROID_ABI STREQUAL "armeabi-v7a")
        add_compile_options(
            -mfpu=neon-vfpv4
            -mfloat-abi=softfp
        )
    endif()

elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    add_compile_options(
        /O2
        /GL
    )
    add_compile_definitions(
        NDEBUG
        GL4ES_SILENCE_MESSAGES
    )
endif()

# ============================================================
# glslang & SPIRV-Cross
# ============================================================

set(
    ENABLE_RTTI
    ON
    CACHE BOOL "Enables RTTI (will be used by glslang)"
    FORCE
)

set(
    ENABLE_HLSL
    OFF
    CACHE BOOL "Disable HLSL input for glslang"
    FORCE
)

set(
    BUILD_EXTERNAL
    OFF
    CACHE BOOL "Disable external dependencies for glslang"
    FORCE
)

set(
    ENABLE_GLSLANG_BINARIES
    OFF
    CACHE BOOL "Disable glslangValidator/spirv-remap"
    FORCE
)

set(
    ENABLE_OPT
    OFF
    CACHE BOOL "Disable SPIRV-Tools opt usage in glslang"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_C_API
    ON
    CACHE BOOL "Enable SPIRV-Cross C API"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_GLSL
    ON
    CACHE BOOL "Enable SPIRV-Cross GLSL backend"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_REFLECT
    OFF
    CACHE BOOL "Disable SPIRV-Cross reflection"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_HLSL
    OFF
    CACHE BOOL "Disable SPIRV-Cross HLSL backend"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_MSL
    OFF
    CACHE BOOL "Disable SPIRV-Cross MSL backend"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_CPP
    OFF
    CACHE BOOL "Disable SPIRV-Cross C++ API target"
    FORCE
)

set(
    SPIRV_CROSS_CLI
    OFF
    CACHE BOOL "Disable SPIRV-Cross CLI binary"
    FORCE
)

set(
    SPIRV_CROSS_STATIC
    ON
    CACHE BOOL "Prefer static SPIRV-Cross libraries"
    FORCE
)

set(
    SPIRV_CROSS_ENABLE_UTIL
    OFF
    CACHE BOOL "Disable SPIRV-Cross util"
    FORCE
)

add_subdirectory(3rdparty/glslang)
add_subdirectory(3rdparty/SPIRV-Cross)

# ============================================================
# Language standards
# ============================================================

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# ============================================================
# Compiler flags
# ============================================================

set(
    CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -fvisibility=hidden"
)

set(
    CMAKE_C_FLAGS
    "${CMAKE_C_FLAGS} -fPIC -DNO_GBM -DDEFAULT_ES=3 -fvisibility=hidden"
)

if(ANDROID)
    set(
        CMAKE_C_FLAGS
        "${CMAKE_C_FLAGS} -DNOX11"
    )
endif()

# ============================================================
# Include directories
# ============================================================

include_directories(
    ${CMAKE_SOURCE_DIR}/include
)

# ============================================================
# C sources
# ============================================================

set(C_SRC_FILES

    src/gl/string_utils.c
    src/gl/cJSON.c
    src/gl/config.c

    src/gl/arbconverter.c
    src/gl/arbgenerator.c
    src/gl/arbhelper.c
    src/gl/arbparser.c

    src/gl/array.c
    src/gl/blend.c
    src/gl/blit.c
    src/gl/buffers.c
    src/gl/build_info.c
    src/gl/debug.c
    src/gl/decompress.c
    src/gl/depth.c
    src/gl/directstate.c
    src/gl/drawing.c
    src/gl/enable.c
    src/gl/envvars.c
    src/gl/eval.c
    src/gl/face.c
    src/gl/fog.c
    src/gl/fpe.c
    src/gl/fpe_cache.c
    src/gl/fpe_shader.c
    src/gl/framebuffers.c

    src/gl/gl_lookup.c
    src/gl/gl4es.c
    src/gl/glstate.c

    src/gl/hint.c
    src/gl/init.c
    src/gl/light.c
    src/gl/line.c
    src/gl/list.c
    src/gl/listdraw.c
    src/gl/listrl.c
    src/gl/loader.c
    src/gl/logs.c

    src/gl/matrix.c
    src/gl/matvec.c

    src/gl/oldprogram.c
    src/gl/pixel.c
    src/gl/planes.c
    src/gl/pointsprite.c
    src/gl/preproc.c
    src/gl/program.c
    src/gl/queries.c
    src/gl/raster.c
    src/gl/render.c
    src/gl/shader.c
    src/gl/shaderconv.c
    src/gl/shader_hacks.c
    src/gl/stack.c
    src/gl/stencil.c
    src/gl/stubs.c

    src/gl/texenv.c
    src/gl/texgen.c
    src/gl/texture.c
    src/gl/texture_compressed.c
    src/gl/texture_params.c
    src/gl/texture_read.c
    src/gl/texture_3d.c

    src/gl/uniform.c
    src/gl/vertexattrib.c

    src/gl/wrap/gl4eswraps.c
    src/gl/wrap/gles.c
    src/gl/wrap/glstub.c
    src/gl/wrap/glesnative.cpp

    src/gl/math/matheval.c

    src/glx/hardext.c
    src/glx/lookup.c
    src/glx/gbm.c
    src/glx/streaming.c

    src/gl/vgpu/shaderconv.c

    src/gl/samplers.c
)

# ============================================================
# Desktop GLX support
# ============================================================

if(NOT ANDROID)
    list(
        APPEND
        C_SRC_FILES
        src/glx/glx.c
    )
endif()

# ============================================================
# C++ sources
# ============================================================

set(CPP_SRC_FILES

    src/gl/glsl/glsl_for_es.cpp
    src/gl/getter.cpp
)

# ============================================================
# All sources
# ============================================================

set(
    SRC_FILES
    ${CPP_SRC_FILES}
    ${C_SRC_FILES}
)

# ============================================================
# NW shared library
# ============================================================

add_library(
    nw
    SHARED
    ${SRC_FILES}
)

# ============================================================
# NW version definitions
# ============================================================

target_compile_definitions(
    nw
    PRIVATE

    NW_BUILD

    NW_VERSION_MAJOR=${PROJECT_VERSION_MAJOR}
    NW_VERSION_MINOR=${PROJECT_VERSION_MINOR}
    NW_VERSION_PATCH=${PROJECT_VERSION_PATCH}
)

# ============================================================
# NW include directories
# ============================================================

target_include_directories(
    nw
    PUBLIC

    ${CMAKE_SOURCE_DIR}/include
)

# ============================================================
# glslang / SPIRV-Cross
# ============================================================

target_link_libraries(
    nw
    PRIVATE

    glslang::glslang
    spirv-cross-c
)

# ============================================================
# Platform-specific libraries
# ============================================================

if(MACOS)

    # iOS / Apple

    target_link_libraries(
        nw
        PUBLIC

        "-F${CMAKE_SOURCE_DIR}/external"
        "-framework libEGL"
        "-framework libGLESv2"
        "-framework Metal"
        "-framework Foundation"
        "-framework QuartzCore"
    )

elseif(ANDROID)

    # Android

    target_link_libraries(
        nw
        PUBLIC

        android
        log
    )

else()

    # Desktop / other platforms
    # No additional platform libraries required here.

endif()

# ============================================================
# NW output configuration
# ============================================================

set_target_properties(
    nw
    PROPERTIES

    OUTPUT_NAME "nw"

    LINK_FLAGS "-Wl,--strip-all"

    POSITION_INDEPENDENT_CODE ON
)

# ============================================================
# Compile commands
# ============================================================

set(
    CMAKE_EXPORT_COMPILE_COMMANDS
    ON
)

# ============================================================
# Build information
# ============================================================

message(STATUS "========================================")
message(STATUS "NW Renderer")
message(STATUS "Version: ${PROJECT_VERSION}")
message(STATUS "Platform: ${CMAKE_SYSTEM_NAME}")
message(STATUS "Architecture: ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "Android: ${ANDROID}")
message(STATUS "Android ABI: ${ANDROID_ABI}")
message(STATUS "Android Platform: ${ANDROID_PLATFORM}")
message(STATUS "C Compiler: ${CMAKE_C_COMPILER}")
message(STATUS "C++ Compiler: ${CMAKE_CXX_COMPILER}")
message(STATUS "========================================")
