# This is not meant to be included by the top-level.
cmake_minimum_required (VERSION 2.8.7)
project(NEOVIM_DEPS)

set(DEPS_INSTALL_DIR "${CMAKE_BINARY_DIR}/usr")
set(DEPS_BIN_DIR "${CMAKE_BINARY_DIR}/usr/bin")
set(DEPS_LIB_DIR "${CMAKE_BINARY_DIR}/usr/lib")
set(DEPS_BUILD_DIR "${CMAKE_BINARY_DIR}/build")
set(DEPS_DOWNLOAD_DIR "${DEPS_BUILD_DIR}/downloads")

option(USE_BUNDLED "Use bundled dependencies." ON)

option(USE_BUNDLED_LIBUNIBILIUM "Use the bundled libunibilium." ${USE_BUNDLED})
option(USE_BUNDLED_LIBTERMKEY "Use the bundled libtermkey." ${USE_BUNDLED})
option(USE_BUNDLED_LIBVTERM "Use the bundled libvterm." ${USE_BUNDLED})
option(USE_BUNDLED_LIBUV "Use the bundled libuv." ${USE_BUNDLED})
option(USE_BUNDLED_MSGPACK "Use the bundled msgpack." ${USE_BUNDLED})
option(USE_BUNDLED_LUAJIT "Use the bundled version of luajit." ${USE_BUNDLED})
option(USE_BUNDLED_LUAROCKS "Use the bundled version of luarocks." ${USE_BUNDLED})

# TODO: add windows support

find_program(MAKE_PRG NAMES gmake make)
if(MAKE_PRG)
  execute_process(
    COMMAND "${MAKE_PRG}" --version
    OUTPUT_VARIABLE MAKE_VERSION_INFO)
  if(NOT "${OUTPUT_VARIABLE}" MATCHES ".*GNU.*")
    unset(MAKE_PRG)
  endif()
endif()
if(NOT MAKE_PRG)
  message(FATAL_ERROR "GNU Make is required to build the dependencies.")
else()
  message(STATUS "Found GNU Make at ${MAKE_PRG}")
endif()

# When using make, use the $(MAKE) variable to avoid warning about the job
# server.
if(CMAKE_GENERATOR MATCHES "Makefiles")
  set(MAKE_PRG "$(MAKE)")
endif()

if(CMAKE_C_COMPILER_ARG1)
  set(DEPS_C_COMPILER "${CMAKE_C_COMPILER} ${CMAKE_C_COMPILER_ARG1}")
else()
  set(DEPS_C_COMPILER "${CMAKE_C_COMPILER}")
endif()

include(ExternalProject)

set(LIBUV_URL https://github.com/libuv/libuv/archive/v1.2.0.tar.gz)
set(LIBUV_SHA256 bebf424bb239867bbf609abad09a256cae7808c9d5cb346b779acd4b97a56693)

set(MSGPACK_URL https://github.com/msgpack/msgpack-c/archive/f6d0cd9a4ba46f4341014a199e3d352fad76b215.tar.gz)
set(MSGPACK_SHA256 988bb2bf86bb0f69816cbcbe2218285b94dbaa27e839f8b1ffdb0b934a7d726a)

set(LUAJIT_URL http://luajit.org/download/LuaJIT-2.0.3.tar.gz)
set(LUAJIT_SHA256 55be6cb2d101ed38acca32c5b1f99ae345904b365b642203194c585d27bebd79)

set(LUAROCKS_URL https://github.com/keplerproject/luarocks/archive/0587afbb5fe8ceb2f2eea16f486bd6183bf02f29.tar.gz)
set(LUAROCKS_SHA256 c8ad50938fed66beba74a73621d14121d4a40b796e01c45238de4cdcb47d5e0b)

set(LIBUNIBILIUM_URL https://github.com/mauke/unibilium/archive/bb979ff6f66a18663e15d086dec6276561b86ee0.tar.gz)
set(LIBUNIBILIUM_SHA256 bec06ea90128b46f28b91b8b52b861dede5f4ede0a92f05178b3c7bcec237dd1)

set(LIBTERMKEY_URL https://github.com/neovim/libtermkey/archive/8c0cb7108cc63218ea19aa898968eede19e19603.tar.gz)
set(LIBTERMKEY_SHA256 21846369081e6c9a0b615f4b3889c4cb809321c5ccc6e6c1640eb138f1590072)

set(LIBVTERM_URL https://github.com/neovim/libvterm/archive/1b745d29d45623aa8d22a7b9288c7b0e331c7088.tar.gz)
set(LIBVTERM_SHA256 3fc75908256c0d158d6c2a32d39f34e86bfd26364f5404b7d9c03bb70cdc3611)


if(USE_BUNDLED_LIBUNIBILIUM)
  ExternalProject_Add(libunibilium
    PREFIX ${DEPS_BUILD_DIR}
    URL ${LIBUNIBILIUM_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/libunibilium
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/libunibilium
      -DURL=${LIBUNIBILIUM_URL}
      -DEXPECTED_SHA256=${LIBUNIBILIUM_SHA256}
      -DTARGET=libunibilium
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND ${MAKE_PRG} CC=${DEPS_C_COMPILER}
                              PREFIX=${DEPS_INSTALL_DIR}
                              CFLAGS=-fPIC
    INSTALL_COMMAND ${MAKE_PRG} PREFIX=${DEPS_INSTALL_DIR} install)
  list(APPEND THIRD_PARTY_DEPS libunibilium)
endif()

if(USE_BUNDLED_LIBTERMKEY)
  ExternalProject_Add(libtermkey
    PREFIX ${DEPS_BUILD_DIR}
    URL ${LIBTERMKEY_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/libtermkey
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/libtermkey
      -DURL=${LIBTERMKEY_URL}
      -DEXPECTED_SHA256=${LIBTERMKEY_SHA256}
      -DTARGET=libtermkey
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND ""
    INSTALL_COMMAND ${MAKE_PRG} CC=${DEPS_C_COMPILER}
                                PREFIX=${DEPS_INSTALL_DIR}
                                PKG_CONFIG_PATH=${DEPS_LIB_DIR}/pkgconfig
                                CFLAGS=-fPIC
                                install)
  list(APPEND THIRD_PARTY_DEPS libtermkey)
  add_dependencies(libtermkey libunibilium)
endif()

if(USE_BUNDLED_LIBVTERM)
  ExternalProject_Add(libvterm
    PREFIX ${DEPS_BUILD_DIR}
    URL ${LIBVTERM_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/libvterm
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/libvterm
      -DURL=${LIBVTERM_URL}
      -DEXPECTED_SHA256=${LIBVTERM_SHA256}
      -DTARGET=libvterm
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND ""
    INSTALL_COMMAND ${MAKE_PRG} CC=${DEPS_C_COMPILER}
                                PREFIX=${DEPS_INSTALL_DIR}
                                CFLAGS=-fPIC
                                install)
  list(APPEND THIRD_PARTY_DEPS libvterm)
endif()


if(USE_BUNDLED_LIBUV)
  ExternalProject_Add(libuv
    PREFIX ${DEPS_BUILD_DIR}
    URL ${LIBUV_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/libuv
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/libuv
      -DURL=${LIBUV_URL}
      -DEXPECTED_SHA256=${LIBUV_SHA256}
      -DTARGET=libuv
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    CONFIGURE_COMMAND sh ${DEPS_BUILD_DIR}/src/libuv/autogen.sh &&
      ${DEPS_BUILD_DIR}/src/libuv/configure --with-pic --disable-shared
        --prefix=${DEPS_INSTALL_DIR} --libdir=${DEPS_INSTALL_DIR}/lib
        CC=${DEPS_C_COMPILER}
    INSTALL_COMMAND ${MAKE_PRG} install)
  list(APPEND THIRD_PARTY_DEPS libuv)
endif()

if(USE_BUNDLED_MSGPACK)
  ExternalProject_Add(msgpack
    PREFIX ${DEPS_BUILD_DIR}
    URL ${MSGPACK_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/msgpack
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/msgpack
      -DURL=${MSGPACK_URL}
      -DEXPECTED_SHA256=${MSGPACK_SHA256}
      -DTARGET=msgpack
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    CONFIGURE_COMMAND cmake ${DEPS_BUILD_DIR}/src/msgpack
       -DMSGPACK_ENABLE_CXX=OFF
       -DMSGPACK_BUILD_TESTS=OFF
       -DCMAKE_INSTALL_PREFIX=${DEPS_INSTALL_DIR}
       -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
       "-DCMAKE_C_FLAGS:STRING=${CMAKE_C_COMPILER_ARG1} -fPIC"
    BUILD_COMMAND ${CMAKE_COMMAND} --build .
    INSTALL_COMMAND ${CMAKE_COMMAND}
      -DREMOVE_FILE_GLOB=${DEPS_INSTALL_DIR}/lib/${CMAKE_SHARED_LIBRARY_PREFIX}msgpack*${CMAKE_SHARED_LIBRARY_SUFFIX}*
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/InstallMsgpack.cmake)
  list(APPEND THIRD_PARTY_DEPS msgpack)
endif()

if(USE_BUNDLED_LUAJIT)
  ExternalProject_Add(luajit
    PREFIX ${DEPS_BUILD_DIR}
    URL ${LUAJIT_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/luajit
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/luajit
      -DURL=${LUAJIT_URL}
      -DEXPECTED_SHA256=${LUAJIT_SHA256}
      -DTARGET=luajit
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND ""
    INSTALL_COMMAND ${MAKE_PRG} CC=${DEPS_C_COMPILER}
                                PREFIX=${DEPS_INSTALL_DIR}
                                CFLAGS=-fPIC
                                CFLAGS+=-DLUAJIT_DISABLE_JIT
                                CFLAGS+=-DLUA_USE_APICHECK
                                CFLAGS+=-DLUA_USE_ASSERT
                                CCDEBUG+=-g
                                BUILDMODE=static
                                install)
  list(APPEND THIRD_PARTY_DEPS luajit)
endif()

if(USE_BUNDLED_LUAROCKS)
  if(USE_BUNDLED_LUAJIT)
    list(APPEND LUAROCKS_OPTS
      --with-lua=${DEPS_INSTALL_DIR}
      --with-lua-include=${DEPS_INSTALL_DIR}/include/luajit-2.0)
  endif()
  ExternalProject_Add(luarocks
    PREFIX ${DEPS_BUILD_DIR}
    URL ${LUAROCKS_URL}
    DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/luarocks
    DOWNLOAD_COMMAND ${CMAKE_COMMAND}
      -DPREFIX=${DEPS_BUILD_DIR}
      -DDOWNLOAD_DIR=${DEPS_DOWNLOAD_DIR}/luarocks
      -DURL=${LUAROCKS_URL}
      -DEXPECTED_SHA256=${LUAROCKS_SHA256}
      -DTARGET=luarocks
      -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DownloadAndExtractFile.cmake
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ${DEPS_BUILD_DIR}/src/luarocks/configure
      --prefix=${DEPS_INSTALL_DIR} --force-config ${LUAROCKS_OPTS}
      --lua-suffix=jit
    BUILD_COMMAND ""
    INSTALL_COMMAND ${MAKE_PRG} bootstrap)
  list(APPEND THIRD_PARTY_DEPS luarocks)
  if(USE_BUNDLED_LUAJIT)
    add_dependencies(luarocks luajit)
  endif()

  add_custom_command(OUTPUT ${DEPS_LIB_DIR}/luarocks/rocks/stable-busted-deps
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build lua_cliargs 2.3-3 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build luafilesystem 1.6.3-1 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build dkjson 2.5-1 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build say 1.2-1 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build luassert 1.7.2-0 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build ansicolors 1.0.2-3 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build penlight 1.0.0-1 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build mediator_lua 1.1-3 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    COMMAND touch ${DEPS_LIB_DIR}/luarocks/rocks/stable-busted-deps
    DEPENDS luarocks)
  add_custom_target(stable-busted-deps
    DEPENDS ${DEPS_LIB_DIR}/luarocks/rocks/stable-busted-deps)

  add_custom_command(OUTPUT ${DEPS_BIN_DIR}/busted
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build busted 2.0.rc4 CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    DEPENDS stable-busted-deps)
  add_custom_target(busted
    DEPENDS ${DEPS_BIN_DIR}/busted)

  # lua-messagepack doesn't depend on busted, but luarocks is unhappy to have
  # two instances running in parallel.  So we depend on busted to force it to
  # be serialized.
  add_custom_command(OUTPUT ${DEPS_LIB_DIR}/luarocks/rocks/lua-messagepack
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build lua-messagepack CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    DEPENDS busted)
  add_custom_target(lua-messagepack
    DEPENDS ${DEPS_LIB_DIR}/luarocks/rocks/lua-messagepack)

  # Like before, depend on lua-messagepack to ensure serialization of install
  # commands
  add_custom_command(OUTPUT ${DEPS_LIB_DIR}/luarocks/rocks/lpeg
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build lpeg CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER}
    DEPENDS lua-messagepack)
  add_custom_target(lpeg
    DEPENDS ${DEPS_LIB_DIR}/luarocks/rocks/lpeg)

  add_custom_command(OUTPUT ${DEPS_LIB_DIR}/luarocks/rocks/nvim-client
    COMMAND ${DEPS_BIN_DIR}/luarocks
    ARGS build https://raw.githubusercontent.com/neovim/lua-client/8cc5b6090ac61cd0bba53ba984f15792fbb64573/nvim-client-0.0.1-11.rockspec CC=${DEPS_C_COMPILER} LD=${DEPS_C_COMPILER} LIBUV_DIR=${DEPS_INSTALL_DIR}
    DEPENDS lpeg libuv)
  add_custom_target(nvim-client
    DEPENDS ${DEPS_LIB_DIR}/luarocks/rocks/nvim-client)

  list(APPEND THIRD_PARTY_DEPS stable-busted-deps busted lua-messagepack lpeg nvim-client)
endif()

add_custom_target(third-party ALL
  COMMAND ${CMAKE_COMMAND} -E touch .third-party
  DEPENDS ${THIRD_PARTY_DEPS})
