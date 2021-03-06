include_directories(..)
include_directories(../../src)
include_directories(../../src/tests)

if (BUILD_TESTING)
  ## reference implementation with simple size-doubling buffer without
  ## jemalloc size tricks
  add_library(doubling_buffer_ftcxx STATIC
    doubling_buffer
    ../cursor
    )
  add_dependencies(doubling_buffer_ftcxx install_tdb_h)

  foreach (impl
      ftcxx
      doubling_buffer_ftcxx
      )
    foreach (with_jemalloc
        ON
        OFF
        )
      foreach (test
          buffer_test
          cursor_test
          )
        set(_testname ${impl}_${test})
        if (with_jemalloc)
          set(_testname ${_testname}_j)
        endif ()
        add_executable(${_testname} ${test})
        if (with_jemalloc)
          if (APPLE)
            target_link_libraries(${_testname} -Wl,-force_load jemalloc)
          else ()
            target_link_libraries(${_testname} -Wl,--whole-archive jemalloc -Wl,--no-whole-archive)
          endif ()
        endif ()
        target_link_libraries(${_testname} ${impl})
        target_link_libraries(${_testname} ${LIBTOKUDB} ${LIBTOKUPORTABILITY})

        add_test(${_testname} ${_testname})
      endforeach ()
    endforeach ()
  endforeach ()
endif (BUILD_TESTING)