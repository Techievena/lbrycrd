cmake_minimum_required(VERSION 3.7)
project(lbrycrd_clion) # Do not use for full compile. This is for CLion syntax checking only. 

set (CMAKE_CXX_STANDARD 98) # 03 is not supported by cmake, but this will soon be 11 (after upstream bitcoin merge)
# I thought that setting the standard would disable the clang-tidy c++11 tips; nope.

if(EXISTS "build/boost")
    set(BOOST_ROOT "build/boost" CACHE PATH "Boost library path")
    set(Boost_NO_SYSTEM_PATHS on CACHE BOOL "Do not search system for Boost")
endif()
find_package(Boost REQUIRED COMPONENTS filesystem program_options thread chrono locale)

file(GLOB sources
        src/*.h src/*.cpp
        src/wallet/*.h src/wallet/*.cpp
        src/support/*.h src/support/*.cpp src/support/allocators/*.h
        src/script/*.h src/script/*.cpp
        src/rpc/*.h src/rpc/*.cpp
        src/primitives/*.h src/primitives/*.cpp
        src/policy/*.h src/policy/*.cpp
        src/crypto/*.h src/crypto/*.cpp
        src/consensus/*.h src/consensus/*.cpp
        src/compat/*.h src/compat/*.cpp
    )
list(FILTER sources EXCLUDE REGEX "src/bitcoin*.cpp$")

include_directories(${Boost_INCLUDE_DIRS}
        build/bdb/include
        build/libevent/include
        build/openssl/include
        src/support/allocators
        src/support
        src/rpc
        src/policy
        src/wallet src/script
        src/leveldb/helpers/memenv
        src/leveldb/include
        src/config
        src/crypto
        src/compat
        src/obj
        src/univalue/include
        src/secp256k1/include
        src/
    )

add_compile_definitions(HAVE_CONFIG_H)

add_executable(lbrycrd-cli src/bitcoin-cli.cpp ${sources})
add_executable(lbrycrd-tx src/bitcoin-tx.cpp ${sources})
add_executable(lbrycrdd src/bitcoind.cpp ${sources})

file(GLOB tests src/test/*.cpp)
foreach(test ${tests})
    get_filename_component(filename ${test} NAME_WE)
    add_executable(${filename} ${test} ${sources})
    target_include_directories(${filename} PRIVATE src/test)
endforeach(test)
