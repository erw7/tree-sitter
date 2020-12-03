cmake_minimum_required(VERSION 2.8.12)
project(tree-sitter LANGUAGES C)

set(tree_sitter_sources
  lib/src/get_changed_ranges.c
  lib/src/language.c
  lib/src/lexer.c
  lib/src/node.c
  lib/src/parser.c
  lib/src/query.c
  lib/src/stack.c
  lib/src/subtree.c
  lib/src/tree.c
  lib/src/tree_cursor.c)

add_library(tree_sitter_shared SHARED ${tree_sitter_sources})
target_include_directories(tree_sitter_shared
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/lib/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/lib/src>)
set_target_properties(tree_sitter_shared PROPERTIES OUTPUT_NAME "tree-sitter")
file(STRINGS Makefile Makefile)
foreach(line IN LISTS Makefile)
  if (line MATCHES "^SONAME_MAJOR +:= ([0-9]+)")
    set(TREE_SITTER_VERSION_MAJOR "${CMAKE_MATCH_1}")
  elseif(line MATCHES "^SONAME_MINOR +:= ([0-9]+)")
    set(TREE_SITTER_VERSION_MINOR "${CMAKE_MATCH_1}")
  elseif(line MATCHES "^SONAME_REVISION +:= ([0-9]+)")
    set(TREE_SITTER_VERSION_REVISION "${CMAKE_MATCH_1}")
  endif()
endforeach(line)
set_target_properties(tree_sitter_shared PROPERTIES
  VERSION ${TREE_SITTER_VERSION_MAJOR}.${TREE_SITTER_VERSION_MINOR}.0
  SOVERSION ${TREE_SITTER_VERSION_MAJOR})

add_library(tree_sitter_static STATIC ${tree_sitter_sources})
target_include_directories(tree_sitter_static
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/lib/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/lib/src>)
set_target_properties(tree_sitter_static PROPERTIES OUTPUT_NAME "tree-sitter")

set(VERSION
  ${TREE_SITTER_VERSION_MAJOR}.${TREE_SITTER_VERSION_MINOR}.${TREE_SITTER_VERSION_REVISION})
set(PREFIX ${CMAKE_INSTALL_PREFIX})
set(LIBDIR ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR})
set(INCLUDEDIR ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR})
configure_file(tree-sitter.pc.in tree-sitter.pc @ONLY)

include(GNUInstallDirs)

install(FILES ${PROJECT_BINARY_DIR}/tree-sitter.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
install(DIRECTORY lib/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(TARGETS tree_sitter_shared tree_sitter_static
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
