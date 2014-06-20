cmake_minimum_required(VERSION 2.8.12.2 FATAL_ERROR)
project(capnproto)


#==================================================================================================#
# For makefile (single-configuration) generators, if it's not already set, make the build type     #
# Release by default                                                                               #
#==================================================================================================#
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  message(STATUS "No build type given.  Setting CMAKE_BUILD_TYPE to Release.")
  set(CMAKE_BUILD_TYPE Release)
endif()


#==================================================================================================#
# Try to avoid "in-source" builds.  See                                                            #
# http://www.cmake.org/Wiki/CMake_FAQ#Out-of-source_build_trees                                    #
#==================================================================================================#
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)


#==================================================================================================#
# Some boilerplate settings                                                                        #
#==================================================================================================#
set(CMAKE_ERROR_DEPRECATED ON)
set(CMAKE_ERROR_ON_ABSOLUTE_INSTALL_DESTINATION ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
if(UNIX)
  set(CMAKE_THREAD_PREFER_PTHREAD true)
  find_package(Threads REQUIRED)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lpthread")
endif()


#==================================================================================================#
# User-selectable options                                                                          #
#==================================================================================================#
option(BUILD_SHARED_LIBS "All libraries will be built shared, not static." OFF)
set(CAPNP "" CACHE FILEPATH "Path to external copy of Capnp compiler")


#==================================================================================================#
# kj library                                                                                       #
#==================================================================================================#
set(IncludeKjHeaders
  c++/src/kj/common.h
  c++/src/kj/units.h
  c++/src/kj/memory.h
  c++/src/kj/refcount.h
  c++/src/kj/array.h
  c++/src/kj/vector.h
  c++/src/kj/string.h
  c++/src/kj/string-tree.h
  c++/src/kj/exception.h
  c++/src/kj/debug.h
  c++/src/kj/arena.h
  c++/src/kj/io.h
  c++/src/kj/tuple.h
  c++/src/kj/one-of.h
  c++/src/kj/function.h
  c++/src/kj/mutex.h
  c++/src/kj/thread.h
  c++/src/kj/threadlocal.h
  c++/src/kj/time.h
  c++/src/kj/main.h
  )

set(IncludeKjParseHeaders
  c++/src/kj/parse/common.h
  c++/src/kj/parse/char.h
  )

set(KjSources
  c++/src/kj/common.c++
  c++/src/kj/units.c++
  c++/src/kj/memory.c++
  c++/src/kj/refcount.c++
  c++/src/kj/array.c++
  c++/src/kj/string.c++
  c++/src/kj/string-tree.c++
  c++/src/kj/exception.c++
  c++/src/kj/debug.c++
  c++/src/kj/arena.c++
  c++/src/kj/io.c++
  c++/src/kj/mutex.c++
  c++/src/kj/thread.c++
  c++/src/kj/main.c++
  c++/src/kj/parse/char.c++
  )

add_library(kj ${IncludeKjHeaders} ${IncludeKjParseHeaders} ${KjSources})

target_compile_definitions(kj PUBLIC $<$<BOOL:${MSVC}>:KJ_NO_COMPILER_CHECK>)

target_compile_options(kj PUBLIC $<$<BOOL:${UNIX}>:-std=c++11>)

target_include_directories(kj PUBLIC c++ c++/src)


#==================================================================================================#
# kj-async library                                                                                 #
#==================================================================================================#
set(IncludeKjAsyncHeaders
  c++/src/kj/async-prelude.h
  c++/src/kj/async.h
  c++/src/kj/async-inl.h
  c++/src/kj/async-unix.h
  c++/src/kj/async-io.h
  )

set(KjAsyncSources
  c++/src/kj/async.c++
  c++/src/kj/async-unix.c++
  c++/src/kj/async-io.c++
  )

add_library(kj-async ${IncludeKjAsyncHeaders} ${KjAsyncSources})

target_link_libraries(kj-async kj)


#==================================================================================================#
# capnp library                                                                                    #
#==================================================================================================#
set(IncludeCapnpHeaders
  c++/src/capnp/c++.capnp.h
  c++/src/capnp/common.h
  c++/src/capnp/blob.h
  c++/src/capnp/endian.h
  c++/src/capnp/layout.h
  c++/src/capnp/orphan.h
  c++/src/capnp/list.h
  c++/src/capnp/any.h
  c++/src/capnp/message.h
  c++/src/capnp/schema.capnp.h
  c++/src/capnp/schema.h
  c++/src/capnp/schema-loader.h
  c++/src/capnp/schema-parser.h
  c++/src/capnp/dynamic.h
  c++/src/capnp/pretty-print.h
  c++/src/capnp/serialize.h
  c++/src/capnp/serialize-packed.h
  c++/src/capnp/pointer-helpers.h
  c++/src/capnp/generated-header-support.h
  )

set(CapnpSources
  c++/src/capnp/c++.capnp.c++
  c++/src/capnp/blob.c++
  c++/src/capnp/arena.h
  c++/src/capnp/arena.c++
  c++/src/capnp/layout.c++
  c++/src/capnp/list.c++
  c++/src/capnp/any.c++
  c++/src/capnp/message.c++
  c++/src/capnp/schema.capnp.c++
  c++/src/capnp/schema.c++
  c++/src/capnp/schema-loader.c++
  c++/src/capnp/dynamic.c++
  c++/src/capnp/stringify.c++
  c++/src/capnp/serialize.c++
  c++/src/capnp/serialize-packed.c++
  )

add_library(capnp ${IncludeCapnpHeaders} ${CapnpSources})

target_link_libraries(capnp kj)


#==================================================================================================#
# capnp-rpc library                                                                                #
#==================================================================================================#
set(IncludeCapnpRpcHeaders
  c++/src/capnp/serialize-async.h
  c++/src/capnp/capability.h
  c++/src/capnp/rpc-prelude.h
  c++/src/capnp/rpc.h
  c++/src/capnp/rpc-twoparty.h
  c++/src/capnp/rpc.capnp.h
  c++/src/capnp/rpc-twoparty.capnp.h
  c++/src/capnp/ez-rpc.h
  )

set(CapnpRpcSources
  c++/src/capnp/serialize-async.c++
  c++/src/capnp/capability.c++
  c++/src/capnp/dynamic-capability.c++
  c++/src/capnp/rpc.c++
  c++/src/capnp/rpc.capnp.c++
  c++/src/capnp/rpc-twoparty.c++
  c++/src/capnp/rpc-twoparty.capnp.c++
  c++/src/capnp/ez-rpc.c++
  )

add_library(capnp-rpc ${IncludeCapnpRpcHeaders} ${CapnpRpcSources})

target_link_libraries(capnp-rpc capnp kj-async)


#==================================================================================================#
# capnpc library                                                                                   #
#==================================================================================================#
set(PublicCapnpcInputs
  c++/src/capnp/c++.capnp
  c++/src/capnp/schema.capnp
  c++/src/capnp/rpc.capnp
  c++/src/capnp/rpc-twoparty.capnp
  )

set(CapnpcInputs
  ${PublicCapnpcInputs}
  c++/src/capnp/compiler/lexer.capnp
  c++/src/capnp/compiler/grammar.capnp
  )

set(CapnpcOutputs
  c++/src/capnp/c++.capnp.c++
  c++/src/capnp/c++.capnp.h
  c++/src/capnp/schema.capnp.c++
  c++/src/capnp/schema.capnp.h
  c++/src/capnp/rpc.capnp.c++
  c++/src/capnp/rpc.capnp.h
  c++/src/capnp/rpc-twoparty.capnp.c++
  c++/src/capnp/rpc-twoparty.capnp.h
  c++/src/capnp/compiler/lexer.capnp.c++
  c++/src/capnp/compiler/lexer.capnp.h
  c++/src/capnp/compiler/grammar.capnp.c++
  c++/src/capnp/compiler/grammar.capnp.h
  )

set(CapnpcHeaders
  c++/src/capnp/compiler/md5.h
  c++/src/capnp/compiler/error-reporter.h
  c++/src/capnp/compiler/lexer.capnp.h
  c++/src/capnp/compiler/lexer.h
  c++/src/capnp/compiler/grammar.capnp.h
  c++/src/capnp/compiler/parser.h
  c++/src/capnp/compiler/node-translator.h
  c++/src/capnp/compiler/compiler.h
  )

set(CapnpcSources
  c++/src/capnp/compiler/md5.c++
  c++/src/capnp/compiler/error-reporter.c++
  c++/src/capnp/compiler/lexer.capnp.c++
  c++/src/capnp/compiler/lexer.c++
  c++/src/capnp/compiler/grammar.capnp.c++
  c++/src/capnp/compiler/parser.c++
  c++/src/capnp/compiler/node-translator.c++
  c++/src/capnp/compiler/compiler.c++
  c++/src/capnp/schema-parser.c++
  )

add_library(capnpc ${CapnpcHeaders} ${CapnpcSources} ${CapnpcInputs})

target_link_libraries(capnpc capnp)


#==================================================================================================#
# capnp binary                                                                                     #
#==================================================================================================#
add_executable(capnp-binary
  c++/src/capnp/compiler/module-loader.h
  c++/src/capnp/compiler/module-loader.c++
  c++/src/capnp/compiler/capnp.c++
  )

set_target_properties(capnp-binary PROPERTIES RUNTIME_OUTPUT_NAME capnp)

target_link_libraries(capnp-binary capnpc)


#==================================================================================================#
# capnpc-capnp binary                                                                              #
#==================================================================================================#
add_executable(capnpc-capnp-binary c++/src/capnp/compiler/capnpc-capnp.c++)

set_target_properties(capnpc-capnp-binary PROPERTIES RUNTIME_OUTPUT_NAME capnpc-capnp)

target_link_libraries(capnpc-capnp-binary capnp)


#==================================================================================================#
# capnpc-c++ binary                                                                                #
#==================================================================================================#
add_executable(capnpc-c++-binary c++/src/capnp/compiler/capnpc-c++.c++)

set_target_properties(capnpc-c++-binary PROPERTIES RUNTIME_OUTPUT_NAME capnpc-c++)

target_link_libraries(capnpc-c++-binary capnp)


#==================================================================================================#
# Download and compile GTest                                                                       #
#==================================================================================================#
# Enable ExternalProject CMake module
include(ExternalProject)

# Set default ExternalProject root directory
set_directory_properties(PROPERTIES EP_PREFIX ${CMAKE_BINARY_DIR}/ThirdParty)

# Add gtest
ExternalProject_Add(
    googletest
    SVN_REPOSITORY http://googletest.googlecode.com/svn/tags/release-1.7.0
    TIMEOUT 60
    # Force separate output paths for debug and release builds to allow easy
    # identification of correct lib in subsequent target_link_libraries commands
    CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
               -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG:PATH=DebugLibs
               -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE:PATH=ReleaseLibs
               -Dgtest_force_shared_crt=ON
    # Disable install step
    INSTALL_COMMAND ""
    # Wrap download, configure and build steps in a script to log output
    LOG_DOWNLOAD ON
    LOG_CONFIGURE ON
    LOG_BUILD ON)

# Specify include dir
ExternalProject_Get_Property(googletest source_dir)
set(GTestIncludePath ${source_dir}/include)

# Specify libraries
ExternalProject_Get_Property(googletest binary_dir)
if(MSVC)
  set(Suffix ".lib")
else()
  set(Suffix ".a")
endif()
set(GTestLibs
  debug ${binary_dir}/DebugLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${Suffix}
  optimized ${binary_dir}/ReleaseLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${Suffix}
  debug ${binary_dir}/DebugLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${Suffix}
  optimized ${binary_dir}/ReleaseLibs/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${Suffix}
  )


#==================================================================================================#
# Run capnp-binary against input files                                                             #
#==================================================================================================#
set(TestCapnpcInputs
  ${CMAKE_SOURCE_DIR}/c++/src/capnp/test.capnp
  ${CMAKE_SOURCE_DIR}/c++/src/capnp/test-import.capnp
  ${CMAKE_SOURCE_DIR}/c++/src/capnp/test-import2.capnp
  )

# Output to the build tree to avoid polluting the source tree
set(TestCapnpcOutputs
  ${CMAKE_BINARY_DIR}/test_outputs/capnp/test.capnp.c++
  ${CMAKE_BINARY_DIR}/test_outputs/capnp/test.capnp.h
  ${CMAKE_BINARY_DIR}/test_outputs/capnp/test-import.capnp.c++
  ${CMAKE_BINARY_DIR}/test_outputs/capnp/test-import.capnp.h
  ${CMAKE_BINARY_DIR}/test_outputs/capnp/test-import2.capnp.c++
  ${CMAKE_BINARY_DIR}/test_outputs/capnp/test-import2.capnp.h
  )

if(EXISTS "${CAPNP}")
  set(CapnpCommand "${CAPNP}")
else()
  set(CapnpCommand capnp-binary)
endif()

add_custom_command(OUTPUT ${TestCapnpcOutputs}
                   COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/test_outputs
                   COMMAND ${CapnpCommand} compile --src-prefix=${CMAKE_SOURCE_DIR}/c++/src
                        -o$<TARGET_FILE:capnpc-c++-binary>:${CMAKE_BINARY_DIR}/test_outputs
                        -I${CMAKE_SOURCE_DIR}/c++/src ${TestCapnpcInputs}
                   COMMENT "Running capnp...")


#==================================================================================================#
# capnp-test binary                                                                                #
#==================================================================================================#
set(CapnpTestSources
  c++/src/kj/common-test.c++
  c++/src/kj/memory-test.c++
  c++/src/kj/refcount-test.c++
  c++/src/kj/array-test.c++
  c++/src/kj/string-test.c++
  c++/src/kj/string-tree-test.c++
  c++/src/kj/exception-test.c++
  c++/src/kj/debug-test.c++
  c++/src/kj/arena-test.c++
  c++/src/kj/units-test.c++
  c++/src/kj/tuple-test.c++
  c++/src/kj/one-of-test.c++
  c++/src/kj/function-test.c++
  c++/src/kj/io-test.c++
  c++/src/kj/mutex-test.c++
  c++/src/kj/threadlocal-test.c++
  c++/src/kj/threadlocal-pthread-test.c++
  c++/src/kj/async-test.c++
  c++/src/kj/async-unix-test.c++
  c++/src/kj/async-io-test.c++
  c++/src/kj/parse/common-test.c++
  c++/src/kj/parse/char-test.c++
  c++/src/capnp/common-test.c++
  c++/src/capnp/blob-test.c++
  c++/src/capnp/endian-test.c++
  c++/src/capnp/endian-fallback-test.c++
  c++/src/capnp/endian-reverse-test.c++
  c++/src/capnp/layout-test.c++
  c++/src/capnp/any-test.c++
  c++/src/capnp/message-test.c++
  c++/src/capnp/capability-test.c++
  c++/src/capnp/schema-test.c++
  c++/src/capnp/schema-loader-test.c++
  c++/src/capnp/dynamic-test.c++
  c++/src/capnp/stringify-test.c++
  c++/src/capnp/encoding-test.c++
  c++/src/capnp/orphan-test.c++
  c++/src/capnp/serialize-test.c++
  c++/src/capnp/serialize-async-test.c++
  c++/src/capnp/serialize-packed-test.c++
  c++/src/capnp/rpc-test.c++
  c++/src/capnp/rpc-twoparty-test.c++
  c++/src/capnp/ez-rpc-test.c++
  c++/src/capnp/test-util.c++
  c++/src/capnp/test-util.h
  c++/src/capnp/compiler/lexer-test.c++
  c++/src/capnp/compiler/md5-test.c++
  )

add_executable(capnp-test ${CapnpTestSources} ${TestCapnpcOutputs})

target_include_directories(capnp-test PRIVATE ${GTestIncludePath} ${CMAKE_BINARY_DIR}/test_outputs)

target_link_libraries(capnp-test ${GTestLibs} capnpc capnp-rpc)

add_dependencies(capnp-test googletest)


#==================================================================================================#
# capnp-evolution-test binary                                                                      #
#==================================================================================================#
add_executable(capnp-evolution-test c++/src/capnp/compiler/evolution-test.c++)

target_link_libraries(capnp-evolution-test capnpc)


#==================================================================================================#
# Register tests with CTest                                                                        #
#==================================================================================================#
include(CTest)
add_test(NAME capnp-test COMMAND capnp-test)
add_test(NAME capnp-evolution-test COMMAND capnp-evolution-test)
if(UNIX)
  # TODO - convert shell script to CMake commands for portability
  add_test(NAME AuxiliaryCommands COMMAND ${CMAKE_SOURCE_DIR}/c++/src/capnp/compiler/capnp-test.sh)
endif()


#==================================================================================================#
# Installer                                                                                        #
#==================================================================================================#
# TODO
