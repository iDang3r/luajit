# Building tools for developing with uJIT.
# Major portions taken verbatim or adapted from the uJIT.
# Copyright (C) 2015-2019 IPONWEB Ltd.

# See the rationale in the root CMakeLists.txt.
cmake_minimum_required(VERSION 3.1 FATAL_ERROR)

set(LUAJIT_TOOLS_DEPS)

if(UJIT_DISABLE_MEMPROF)
  message(STATUS "LuaJIT memory profiler support is disabled")
else()
  # XXX: Can use genex here since the value need to be evaluated
  # on the configuration phase. Fortunately, we know the exact
  # path where LuaJIT binary is located.
  set(LUAJIT_TOOLS_BIN ${LUAJIT_BINARY_DIR}/${LUAJIT_CLI_NAME})
  set(LUAJIT_TOOLS_DIR ${CMAKE_CURRENT_SOURCE_DIR})
  configure_file(luajit-parse-memprof.in luajit-parse-memprof @ONLY ESCAPE_QUOTES)

  add_custom_target(tools-parse-memprof EXCLUDE_FROM_ALL DEPENDS
    luajit-parse-memprof
    memprof/humanize.lua
    memprof/parse.lua
    memprof.lua
    utils/bufread.lua
    utils/symtab.lua
  )
  list(APPEND LUAJIT_TOOLS_DEPS tools-parse-memprof)

  install(FILES
      ${CMAKE_CURRENT_SOURCE_DIR}/memprof/humanize.lua
      ${CMAKE_CURRENT_SOURCE_DIR}/memprof/parse.lua
    DESTINATION ${LUAJIT_DATAROOTDIR}/memprof
    PERMISSIONS
      OWNER_READ OWNER_WRITE
      GROUP_READ
      WORLD_READ
    COMPONENT tools-parse-memprof
  )
  install(FILES
      ${CMAKE_CURRENT_SOURCE_DIR}/utils/bufread.lua
      ${CMAKE_CURRENT_SOURCE_DIR}/utils/symtab.lua
    DESTINATION ${LUAJIT_DATAROOTDIR}/utils
    PERMISSIONS
      OWNER_READ OWNER_WRITE
      GROUP_READ
      WORLD_READ
    COMPONENT tools-parse-memprof
  )
  install(FILES
      ${CMAKE_CURRENT_SOURCE_DIR}/memprof.lua
    DESTINATION ${LUAJIT_DATAROOTDIR}
    PERMISSIONS
      OWNER_READ OWNER_WRITE
      GROUP_READ
      WORLD_READ
    COMPONENT tools-parse-memprof
  )
  install(CODE
    # XXX: Since the auxiliary script need to be configured in
    # other way it need to be reconfigured it prior to its
    # installation. Unfortunately, we need to manually specify
    # the installation path in <configure_file> command.
    # Hope this script will be gone as a result of the issue below
    # https://github.com/tarantool/tarantool/issues/5688.
    "
      set(LUAJIT_TOOLS_BIN ${CMAKE_INSTALL_PREFIX}/bin/${LUAJIT_CLI_NAME})
      set(LUAJIT_TOOLS_DIR ${CMAKE_INSTALL_PREFIX}/${LUAJIT_DATAROOTDIR})
      configure_file(${CMAKE_CURRENT_SOURCE_DIR}/luajit-parse-memprof.in
        ${CMAKE_INSTALL_PREFIX}/bin/luajit-parse-memprof @ONLY ESCAPE_QUOTES)
      message(STATUS \"Installing: ${CMAKE_INSTALL_PREFIX}/bin/luajit-parse-memprof\")
    "
    COMPONENT tools-parse-memprof
  )
endif()

add_custom_target(LuaJIT-tools DEPENDS ${LUAJIT_TOOLS_DEPS})
