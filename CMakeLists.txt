# BoringSSL, following Chromium's third_party/boringssl/BUILD.gn: one library
# of crypto, ssl and pki, plus the assembly (all platforms' .S files are listed;
# each assembles to nothing except on its own platform). The source lists come
# from gen/sources.cmake, as gn's come from gen/sources.gni. Windows NASM
# assembly isn't supported.
cmake_minimum_required(VERSION 3.22)
project(boringssl C CXX ASM)

include(gen/sources.cmake)

add_library(boringssl STATIC
  ${BCM_SOURCES}
  ${CRYPTO_SOURCES}
  ${SSL_SOURCES}
  ${PKI_SOURCES}
  ${BCM_SOURCES_ASM}
  ${CRYPTO_SOURCES_ASM}
)
# external_config.
target_include_directories(boringssl PUBLIC include)
# component_config and internal_config.
target_compile_definitions(boringssl PRIVATE BORINGSSL_IMPLEMENTATION OPENSSL_SMALL)
