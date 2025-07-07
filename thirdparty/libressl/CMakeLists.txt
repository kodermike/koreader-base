list(APPEND PATCH_FILES cmake_update.patch)
list(APPEND PATCH_FILES configure_ac_update.patch)
list(APPEND PATCH_FILES crypto_cmakelist_update.patch)
list(APPEND PATCH_FILES crypto_makefile_update.patch)
list(APPEND PATCH_FILES crypto_elf_x86_update.patch)
list(APPEND PATCH_FILES crypto_macos_x86_make_update.patch)
list(APPEND PATCH_FILES crypto_cpu_caps_darwin.patch)
list(APPEND PATCH_FILES crypto_arch64_linux_cpu_caps.patch)
list(APPEND PATCH_FILES crypto_cpu_caps_none.patch)
list(APPEND PATCH_FILES makefile_ssl.patch)
list(APPEND PATCH_FILES tls_makefile.patch)
if(ANDROID)
    list(APPEND PATCH_FILES android.patch)
endif()

FetchContent_Declare(
  libressl-autogen
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        703bd9caab50b139428cea1aaff9974ebee5742e # release-1.10.0
)
FetchContent_Declare(
  myCompanyIcons
  URL      https://raw.githubusercontent.com/libressl/portable/master/autogen.sh
  URL_HASH MD5=5588a7b18261c20068beabfb4f530b87
)

FetchContent_MakeAvailable(googletest myCompanyIcons)

set(LIBRESSL_AUTOGEN ${OUTPUT_DIR})

add_custom_command(
    OUTPUT ${LIBRESSL_AUTOGEN}
    MAIN_DEPENDENCY autogen.sh
    COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/autogen.sh
)

add_custom_target(libressl ALL DEPENDS ${LIBRESSL_AUTOGEN})

# am__api_version='1.16'
# aclocal --version|head -1|awk -F" " '{print $4}'

list(APPEND CMAKE_ARGS
    -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
    -DBUILD_SHARED_LIBS=$<NOT:$<BOOL:${MONOLIBTIC}>>
    # Project options.
    -DLIBRESSL_APPS=OFF
    -DLIBRESSL_TESTS=OFF
    -DENABLE_ASM=${WANT_SIMD}
)

list(APPEND BUILD_CMD COMMAND ninja)

list(APPEND INSTALL_CMD COMMAND ${CMAKE_COMMAND} --install .)

if(NOT MONOLIBTIC)
    append_shared_lib_install_commands(INSTALL_CMD crypto VERSION 56)
    append_shared_lib_install_commands(INSTALL_CMD ssl VERSION 59)
endif()

external_project(
    DOWNLOAD URL 18079dcf72a398d8c188e67e30c1dc13
    https://github.com/libressl/portable/releases/download/v4.1.0/libressl-4.1.0.tar.gz
    https://cdn.openbsd.org/pub/OpenBSD/LibreSSL/libressl-4.1.0.tar.gz
    PATCH_FILES ${PATCH_FILES}
    CMAKE_ARGS ${CMAKE_ARGS}
    BUILD_COMMAND ${BUILD_CMD}
    INSTALL_COMMAND ${INSTALL_CMD}
)
