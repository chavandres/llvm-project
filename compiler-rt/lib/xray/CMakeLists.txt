# Build for all components of the XRay runtime support library.

# XRay runtime library implementation files.
set(XRAY_SOURCES
  xray_buffer_queue.cpp
  xray_init.cpp
  xray_flags.cpp
  xray_interface.cpp
  xray_log_interface.cpp
  xray_utils.cpp
  )

set(XRAY_DSO_SOURCES
  xray_dso_init.cpp
  )

# Implementation files for all XRay modes.
set(XRAY_FDR_MODE_SOURCES
  xray_fdr_flags.cpp
  xray_fdr_logging.cpp
  )

set(XRAY_BASIC_MODE_SOURCES
  xray_basic_flags.cpp
  xray_basic_logging.cpp
  )

set(XRAY_PROFILING_MODE_SOURCES
  xray_profile_collector.cpp
  xray_profiling.cpp
  xray_profiling_flags.cpp
  )

# Implementation files for all XRay architectures.
set(x86_64_SOURCES
  xray_x86_64.cpp
  xray_trampoline_x86_64.S
  )

set(x86_64_DSO_SOURCES
   xray_trampoline_x86_64.S
   )


set(arm_SOURCES
  xray_arm.cpp
  xray_trampoline_arm.S
  )

set(armhf_SOURCES
  ${arm_SOURCES}
  )

set(aarch64_SOURCES
  xray_AArch64.cpp
  xray_trampoline_AArch64.S
  )

set(aarch64_DSO_SOURCES
  xray_trampoline_AArch64.S
  )

set(loongarch64_SOURCES
  xray_loongarch64.cpp
  xray_trampoline_loongarch64.S
  )

set(mips_SOURCES
  xray_mips.cpp
  xray_trampoline_mips.S
  )

set(mipsel_SOURCES
  xray_mips.cpp
  xray_trampoline_mips.S
  )

set(mips64_SOURCES
  xray_mips64.cpp
  xray_trampoline_mips64.S
  )

set(mips64el_SOURCES
  xray_mips64.cpp
  xray_trampoline_mips64.S
  )

set(powerpc64le_SOURCES
  xray_powerpc64.cpp
  xray_trampoline_powerpc64.cpp
  xray_trampoline_powerpc64_asm.S
  )

set(hexagon_SOURCES
  xray_hexagon.cpp
  xray_trampoline_hexagon.S
  )

set(riscv32_SOURCES
  xray_riscv.cpp
  xray_trampoline_riscv32.S
  )

set(riscv64_SOURCES
  xray_riscv.cpp
  xray_trampoline_riscv64.S
  )

  set(s390x_SOURCES
  xray_s390x.cpp
  xray_trampoline_s390x.S
  )
# Enable vector instructions in the assembly file.
set_source_files_properties(xray_trampoline_s390x.S PROPERTIES COMPILE_FLAGS -march=z13)

set(XRAY_SOURCE_ARCHS
  arm
  armhf
  aarch64
  hexagon
  mips
  mipsel
  mips64
  mips64el
  powerpc64le
  s390x
  x86_64
  )

set(XRAY_IMPL_HEADERS
  xray_allocator.h
  xray_basic_flags.h
  xray_basic_flags.inc
  xray_basic_logging.h
  xray_buffer_queue.h
  xray_defs.h
  xray_fdr_controller.h
  xray_fdr_flags.h
  xray_fdr_flags.inc
  xray_fdr_log_records.h
  xray_fdr_log_writer.h
  xray_fdr_logging.h
  xray_flags.h
  xray_flags.inc
  xray_function_call_trie.h
  xray_interface_internal.h
  xray_powerpc64.inc
  xray_profile_collector.h
  xray_profiling_flags.h
  xray_profiling_flags.inc
  xray_recursion_guard.h
  xray_segmented_array.h
  xray_tsc.h
  xray_utils.h
  xray_x86_64.inc
  )

# Create list of all source files for
# consumption by tests.
set(XRAY_ALL_SOURCE_FILES
  ${XRAY_SOURCES}
  ${XRAY_DSO_SOURCES}
  ${XRAY_FDR_MODE_SOURCES}
  ${XRAY_BASIC_MODE_SOURCES}
  ${XRAY_PROFILING_MODE_SOURCES}
  ${x86_64_SOURCES}
  ${x86_64_DSO_SOURCES}
  ${arm_SOURCES}
  ${armhf_SOURCES}
  ${hexagon_SOURCES}
  ${loongarch64_SOURCES}
  ${mips_SOURCES}
  ${mipsel_SOURCES}
  ${mips64_SOURCES}
  ${mips64el_SOURCES}
  ${powerpc64le_SOURCES}
  ${riscv32_SOURCES}
  ${riscv64_SOURCES}
  ${s390x_SOURCES}
  ${XRAY_IMPL_HEADERS}
  )
list(REMOVE_DUPLICATES XRAY_ALL_SOURCE_FILES)
# Make list that uses absolute paths
set(XRAY_ALL_SOURCE_FILES_ABS_PATHS "")
foreach (src_file ${XRAY_ALL_SOURCE_FILES})
  list(APPEND
    XRAY_ALL_SOURCE_FILES_ABS_PATHS
    "${CMAKE_CURRENT_SOURCE_DIR}/${src_file}")
endforeach()


# Now put it all together...
include_directories(..)
include_directories(../../include)

set(XRAY_CFLAGS
  ${COMPILER_RT_COMMON_CFLAGS}
  ${COMPILER_RT_CXX_CFLAGS})
set(XRAY_COMMON_DEFINITIONS SANITIZER_COMMON_NO_REDEFINE_BUILTINS XRAY_HAS_EXCEPTIONS=1)

# DSO trampolines need to be compiled with GOT addressing
set(XRAY_COMMON_DEFINITIONS_DSO ${XRAY_COMMON_DEFINITIONS} XRAY_PIC)

# Too many existing bugs, needs cleanup.
append_list_if(COMPILER_RT_HAS_WNO_FORMAT -Wno-format XRAY_CFLAGS)

# We don't need RTTI in XRay, so turn that off.
append_rtti_flag(OFF XRAY_CFLAGS)

set(XRAY_LINK_FLAGS ${COMPILER_RT_COMMON_LINK_FLAGS})
set(XRAY_LINK_LIBS
  ${COMPILER_RT_UNWINDER_LINK_LIBS}
  ${COMPILER_RT_CXX_LINK_LIBS})

append_list_if(
  COMPILER_RT_HAS_XRAY_COMPILER_FLAG XRAY_SUPPORTED=1 XRAY_COMMON_DEFINITIONS)
append_list_if(
  COMPILER_RT_BUILD_XRAY_NO_PREINIT XRAY_NO_PREINIT XRAY_COMMON_DEFINITIONS)

add_compiler_rt_component(xray)

set(XRAY_COMMON_RUNTIME_OBJECT_LIBS
    RTSanitizerCommon
    RTSanitizerCommonLibc)

# XRay uses C++ standard library headers.
if (TARGET cxx-headers OR HAVE_LIBCXX)
  set(XRAY_DEPS cxx-headers)
endif()

if (APPLE)
  add_weak_symbols("sanitizer_common" WEAK_SYMBOL_LINK_FLAGS)
  add_weak_symbols("xray" WEAK_SYMBOL_LINK_FLAGS)

  add_compiler_rt_object_libraries(RTXray
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    SOURCES ${XRAY_SOURCES}
    ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    DEPS ${XRAY_DEPS})
  add_compiler_rt_object_libraries(RTXrayDSO
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_DSO_SUPPORTED_ARCH}
    SOURCES ${XRAY_DSO_SOURCES}
    ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS_DSO}
    DEPS ${XRAY_DEPS})
  set(XRAY_RTXRAY_ARCH_LIBS "")
  set(XRAY_DSO_RTXRAY_ARCH_LIBS "")
  foreach(arch ${XRAY_SUPPORTED_ARCH})
    if(NOT ${arch} IN_LIST XRAY_SOURCE_ARCHS)
      continue()
    endif()
    add_compiler_rt_object_libraries(RTXray_${arch}
      OS ${XRAY_SUPPORTED_OS}
      ARCHS ${arch}
      SOURCES ${${arch}_SOURCES}
      ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
      CFLAGS ${XRAY_CFLAGS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      DEPS ${XRAY_DEPS})
    list(APPEND XRAY_RTXRAY_ARCH_LIBS RTXray_${arch})
    if (${arch} IN_LIST XRAY_DSO_SUPPORTED_ARCH)
      add_compiler_rt_object_libraries(RTXrayDSO_${arch}
        OS ${XRAY_SUPPORTED_OS}
        ARCHS ${arch}
        SOURCES ${${arch}_DSO_SOURCES}
        ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
        CFLAGS ${XRAY_CFLAGS}
        DEFS ${XRAY_COMMON_DEFINITIONS_DSO}
        DEPS ${XRAY_DEPS})
      list(APPEND XRAY_DSO_RTXRAY_ARCH_LIBS RTXrayDSO_${arch})
    endif()
  endforeach()
  add_compiler_rt_object_libraries(RTXrayFDR
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    SOURCES ${XRAY_FDR_MODE_SOURCES}
    ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    DEPS ${XRAY_DEPS})
  add_compiler_rt_object_libraries(RTXrayBASIC
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    SOURCES ${XRAY_BASIC_MODE_SOURCES}
    ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    DEPS ${XRAY_DEPS})
  add_compiler_rt_object_libraries(RTXrayPROFILING
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    SOURCES ${XRAY_PROFILING_MODE_SOURCES}
    ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    DEPS ${XRAY_DEPS})

  # We only support running on osx for now.
  add_compiler_rt_runtime(clang_rt.xray
    STATIC
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    OBJECT_LIBS ${XRAY_COMMON_RUNTIME_OBJECT_LIBS} RTXray ${XRAY_RTXRAY_ARCH_LIBS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    LINK_FLAGS ${XRAY_LINK_FLAGS} ${WEAK_SYMBOL_LINK_FLAGS}
    LINK_LIBS ${XRAY_LINK_LIBS}
    PARENT_TARGET xray)
  add_compiler_rt_runtime(clang_rt.xray-dso
    STATIC
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_DSO_SUPPORTED_ARCH}
    OBJECT_LIBS RTXrayDSO ${XRAY_DSO_RTXRAY_ARCH_LIBS}
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    LINK_FLAGS ${XRAY_LINK_FLAGS} ${WEAK_SYMBOL_LINK_FLAGS}
    LINK_LIBS ${XRAY_LINK_LIBS}
    PARENT_TARGET xray)

  add_compiler_rt_runtime(clang_rt.xray-fdr
    STATIC
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    OBJECT_LIBS RTXrayFDR
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    LINK_FLAGS ${XRAY_LINK_FLAGS} ${WEAK_SYMBOL_LINK_FLAGS}
    LINK_LIBS ${XRAY_LINK_LIBS}
    PARENT_TARGET xray)
  add_compiler_rt_runtime(clang_rt.xray-basic
    STATIC
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    OBJECT_LIBS RTXrayBASIC
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    LINK_FLAGS ${XRAY_LINK_FLAGS} ${WEAK_SYMBOL_LINK_FLAGS}
    LINK_LIBS ${XRAY_LINK_LIBS}
    PARENT_TARGET xray)
  add_compiler_rt_runtime(clang_rt.xray-profiling
    STATIC
    OS ${XRAY_SUPPORTED_OS}
    ARCHS ${XRAY_SUPPORTED_ARCH}
    OBJECT_LIBS RTXrayPROFILING
    CFLAGS ${XRAY_CFLAGS}
    DEFS ${XRAY_COMMON_DEFINITIONS}
    LINK_FLAGS ${XRAY_LINK_FLAGS} ${WEAK_SYMBOL_LINK_FLAGS}
    LINK_LIBS ${XRAY_LINK_LIBS}
    PARENT_TARGET xray)
else() # not Apple
  foreach(arch ${XRAY_SUPPORTED_ARCH})
    if(NOT CAN_TARGET_${arch})
      continue()
    endif()
    add_compiler_rt_object_libraries(RTXray
      ARCHS ${arch}
      SOURCES ${XRAY_SOURCES} ${${arch}_SOURCES}
      ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
      CFLAGS ${XRAY_CFLAGS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      DEPS ${XRAY_DEPS})
    add_compiler_rt_object_libraries(RTXrayFDR
      ARCHS ${arch}
      SOURCES ${XRAY_FDR_MODE_SOURCES}
      ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
      CFLAGS ${XRAY_CFLAGS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      DEPS ${XRAY_DEPS})
    add_compiler_rt_object_libraries(RTXrayBASIC
      ARCHS ${arch}
      SOURCES ${XRAY_BASIC_MODE_SOURCES}
      ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
      CFLAGS ${XRAY_CFLAGS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      DEPS ${XRAY_DEPS})
    add_compiler_rt_object_libraries(RTXrayPROFILING
      ARCHS ${arch}
      SOURCES ${XRAY_PROFILING_MODE_SOURCES}
      ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
      CFLAGS ${XRAY_CFLAGS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      DEPS ${XRAY_DEPS})

    # Common XRay archive for instrumented binaries.
    add_compiler_rt_runtime(clang_rt.xray
     STATIC
     ARCHS ${arch}
     CFLAGS ${XRAY_CFLAGS}
     LINK_FLAGS ${XRAY_LINK_FLAGS}
     LINK_LIBS ${XRAY_LINK_LIBS}
     DEFS ${XRAY_COMMON_DEFINITIONS}
     OBJECT_LIBS ${XRAY_COMMON_RUNTIME_OBJECT_LIBS} RTXray
     PARENT_TARGET xray)
    # FDR mode runtime archive (addon for clang_rt.xray)
    add_compiler_rt_runtime(clang_rt.xray-fdr
      STATIC
      ARCHS ${arch}
      CFLAGS ${XRAY_CFLAGS}
      LINK_FLAGS ${XRAY_LINK_FLAGS}
      LINK_LIBS ${XRAY_LINK_LIBS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      OBJECT_LIBS RTXrayFDR
      PARENT_TARGET xray)
    # Basic mode runtime archive (addon for clang_rt.xray)
    add_compiler_rt_runtime(clang_rt.xray-basic
      STATIC
      ARCHS ${arch}
      CFLAGS ${XRAY_CFLAGS}
      LINK_FLAGS ${XRAY_LINK_FLAGS}
      LINK_LIBS ${XRAY_LINK_LIBS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      OBJECT_LIBS RTXrayBASIC
      PARENT_TARGET xray)
    # Profiler Mode runtime
    add_compiler_rt_runtime(clang_rt.xray-profiling
      STATIC
      ARCHS ${arch}
      CFLAGS ${XRAY_CFLAGS}
      LINK_FLAGS ${XRAY_LINK_FLAGS}
      LINK_LIBS ${XRAY_LINK_LIBS}
      DEFS ${XRAY_COMMON_DEFINITIONS}
      OBJECT_LIBS RTXrayPROFILING
      PARENT_TARGET xray)

    if (${arch} IN_LIST XRAY_DSO_SUPPORTED_ARCH)
      add_compiler_rt_object_libraries(RTXrayDSO
        ARCHS ${arch}
        SOURCES ${XRAY_DSO_SOURCES} ${${arch}_DSO_SOURCES} 
        ADDITIONAL_HEADERS ${XRAY_IMPL_HEADERS}
        CFLAGS ${XRAY_CFLAGS}
        DEFS ${XRAY_COMMON_DEFINITIONS_DSO}
        DEPS ${XRAY_DEPS})
      # DSO runtime archive
      add_compiler_rt_runtime(clang_rt.xray-dso
        STATIC
        ARCHS ${arch}
        CFLAGS ${XRAY_CFLAGS}
        LINK_FLAGS ${XRAY_LINK_FLAGS}
        LINK_LIBS ${XRAY_LINK_LIBS}
        DEFS ${XRAY_COMMON_DEFINITIONS}
        OBJECT_LIBS RTXrayDSO
        PARENT_TARGET xray)
    endif()
  endforeach()
endif() # not Apple

if(COMPILER_RT_INCLUDE_TESTS)
  add_subdirectory(tests)
endif()
