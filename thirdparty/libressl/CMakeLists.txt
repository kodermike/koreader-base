# Build in source tree.
set(BINARY_DIR ${SOURCE_DIR})

list(APPEND PATCH_FILES autogen.patch)
list(APPEND PATCH_FILES update.sh.patch)
list(APPEND PATCH_FILES dist.sh.patch)
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

file(REMOVE compile configure depcomp install-sh ltmain.sh missing openssl.cnf)

#list(PREPEND CFG_CMD COMMAND sh autogen.sh)
list(PREPEND CFG_CMD COMMAND sh dist.sh)
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
