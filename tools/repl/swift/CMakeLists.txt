# Set the correct rpath to locate libswiftCore.
if (LLDB_BUILD_FRAMEWORK)
  get_target_property(framework_target_dir liblldb LIBRARY_OUTPUT_DIRECTORY)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath \
      -Wl,${framework_target_dir}/LLDB.framework/${LLDB_FRAMEWORK_RESOURCE_DIR}/Swift/macosx")
elseif( CMAKE_SYSTEM_NAME MATCHES "Linux" )
  # Set the correct rpath to locate libswiftCore
  if (CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "ppc64le")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} \
        -Wl,-rpath,${CMAKE_BINARY_DIR}/../swift-linux-powerpc64le/lib${LLVM_LIBDIR_SUFFIX}/swift/linux/powerpc64le")
  else()
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} \
        -Wl,-rpath,${CMAKE_BINARY_DIR}/../swift-linux-x86_64/lib${LLVM_LIBDIR_SUFFIX}/swift/linux")
  endif()
  set(CMAKE_INSTALL_RPATH "$ORIGIN/../lib/swift/linux:${CMAKE_INSTALL_RPATH}")
endif()

add_lldb_tool(repl_swift
  main.c
  )
target_link_libraries(repl_swift PRIVATE ${CMAKE_DL_LIBS})

# The dummy repl executable is a C program, but we always look for a mangled
# swift symbol (corresponding to main). If we build the repl with debug info,
# the debugger looks at the frame language (looking up the compile unit) and gets
# confused.
if(MSVC)
  # disable inline function expansion so that we have a function that we can
  # break upon
  target_compile_options(repl_swift PRIVATE /Ob0)
endif()
if(NOT MSVC)
  set_target_properties(repl_swift PROPERTIES
      COMPILE_FLAGS "-g0")
endif()
