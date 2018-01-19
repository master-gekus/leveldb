cmake_minimum_required(VERSION 3.1)

project(leveldb)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if (MSVC)
    add_definitions(-DNOMINMAX)
endif()

if (MSVC)
    add_definitions(-DLEVELDB_PLATFORM_MSVC=1)
    set(LEVELDB_PORT_SOURCES
        port/port_msvc.cc
        util/env_msvc.cc
        )
    include_directories(port/win)
else(MSVC)
    add_definitions(-DLEVELDB_PLATFORM_POSIX=1)
    link_libraries(pthread)
    set(LEVELDB_PORT_SOURCES
        port/port_posix.cc
        util/env_posix.cc
        )
endif(MSVC)

include_directories(./ include)

add_library(memenv STATIC
    helpers/memenv/memenv.cc
    helpers/memenv/memenv.h
    )

add_library(${PROJECT_NAME} STATIC
    db/builder.cc
    db/c.cc
    db/dbformat.cc
    db/db_impl.cc
    db/db_iter.cc
    db/dumpfile.cc
    db/filename.cc
    db/log_reader.cc
    db/log_writer.cc
    db/memtable.cc
    db/repair.cc
    db/table_cache.cc
    db/version_edit.cc
    db/version_set.cc
    db/write_batch.cc
    table/block_builder.cc
    table/block.cc
    table/filter_block.cc
    table/format.cc
    table/iterator.cc
    table/merger.cc
    table/table_builder.cc
    table/table.cc
    table/two_level_iterator.cc
    util/arena.cc
    util/bloom.cc
    util/cache.cc
    util/coding.cc
    util/comparator.cc
    util/crc32c.cc
    util/env.cc
    util/filter_policy.cc
    util/hash.cc
    util/histogram.cc
    util/logging.cc
    util/options.cc
    util/status.cc
    ${LEVELDB_PORT_SOURCES}
    )
target_link_libraries(${PROJECT_NAME} memenv)

option(LEVELDB_BUILD_TESTS "Build tests" OFF)

if (LEVELDB_BUILD_TESTS)
    enable_testing()

    add_library(testharness OBJECT
      ${CMAKE_CURRENT_SOURCE_DIR}/util/testharness.cc
      )

    macro(ADD_AUTO_TEST test_name)
        add_executable(${ARGV} $<TARGET_OBJECTS:testharness>)
        target_link_libraries(${test_name} ${PROJECT_NAME})
        add_test(NAME ${test_name} COMMAND ${test_name})
        add_custom_command(TARGET ${test_name} POST_BUILD COMMAND ${test_name})
    endmacro()

    ADD_AUTO_TEST(memenv_test helpers/memenv/memenv_test.cc)
    ADD_AUTO_TEST(c_test db/c_test.c)

endif(LEVELDB_BUILD_TESTS)

