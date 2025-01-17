cmake_minimum_required(VERSION 3.5.0 FATAL_ERROR)

project(demumble CXX)

if (UNIX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fno-exceptions -fno-rtti")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")

  if (${CMAKE_GENERATOR} STREQUAL "Ninja")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color")
  elseif(APPLE)
    # 10.9 chosen somewhat arbitrary; it's the first target where clang defaults
    # to libc++ and ld64 defaults to stripping __TEXT,__eh_frame.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mmacosx-version-min=10.9")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-PIC")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -no-pie")
  endif()
# Folks may want to use MSYS2, MINGW, or any 
# such build environment while using Clang on Windows:
elseif(WIN32 AND CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fno-exceptions -fno-rtti")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")

  if (${CMAKE_GENERATOR} STREQUAL "Ninja")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color")
  endif()
elseif(WIN32 AND CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
  # https://gitlab.kitware.com/cmake/cmake/-/issues/20610
  string(REGEX REPLACE "/GR" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  string(REGEX REPLACE "/EHsc" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  add_definitions(-D_HAS_EXCEPTIONS=0)

  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:inline /EHs-c- /GR-")
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)  # The LLVM build sets this.

  # Disable cl.exe warnings that LLVM disables as well.
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4244 /wd4267")

  # This is apparently the simplest way to statically link the CRT in CMake:
  string(TOUPPER "${CMAKE_BUILD_TYPE}" build)
  set(flag_var "CMAKE_CXX_FLAGS_${build}")

  if (${flag_var} MATCHES "/MD")
    string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
  endif()
else()
  message(FATAL_ERROR "unknown compiler, cannot continue build")
endif()

include_directories(third_party/llvm/include)
add_executable(demumble
               demumble.cc
               third_party/llvm/lib/Demangle/Demangle.cpp
               third_party/llvm/lib/Demangle/ItaniumDemangle.cpp
               third_party/llvm/lib/Demangle/DLangDemangle.cpp
               third_party/llvm/lib/Demangle/MicrosoftDemangle.cpp
               third_party/llvm/lib/Demangle/MicrosoftDemangleNodes.cpp
               third_party/llvm/lib/Demangle/RustDemangle.cpp
)
set_target_properties(demumble PROPERTIES CXX_STANDARD 17
                                          CXX_STANDARD_REQUIRED ON)
