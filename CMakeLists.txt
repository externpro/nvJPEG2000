cmake_minimum_required(VERSION 4.3)
project(nvJPEG2000 VERSION 0.10.0)
set(build 49)
set(SHA256_Linux b17ec3141ec50e0c3b796d4f28dae8370d8db3684deb849c87fc7ad52be1cbcb)
set(SHA256_win64 40684ca3f49b1378526c63ddad6552c5865774962f509b4d8a14dcafe740e152)
set(SHA256_aarch64 e38cf5872a92a5f984a0f370ea2d527da4b9166238ee9a5c423b5289d773dba7)
set(CUDA_VER_MAJOR 12)
# https://developer.download.nvidia.com/compute/nvjpeg2000/redist/redistrib_0.10.0.json
string(TOLOWER ${CMAKE_SYSTEM_NAME} os)
set(ver ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}.${build})
set(nvName nvjpeg2k)
set(nvLibs LIBRARIES ${nvName})
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  set(arch x86_64)
  set(ext zip)
  set(sha ${SHA256_win64})
  set(fnShared _0)
  set(dllPath bin)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(ext tar.xz)
  set(fnStatic _static)
  set(dllPath lib)
  if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64|AMD64)$")
    set(arch x86_64)
    set(sha ${SHA256_Linux})
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|ARM64)$")
    set(arch aarch64)
    set(sha ${SHA256_aarch64})
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  # there is no macOS download available
  # https://developer.download.nvidia.com/compute/nvjpeg2000/redist/libnvjpeg_2k/
  set(nvLibs)
endif()
if(nvLibs)
  set(libName ${PROJECT_NAME}::${nvName})
  set(libFileName ${CMAKE_STATIC_LIBRARY_PREFIX}${nvName}${fnStatic}${CMAKE_STATIC_LIBRARY_SUFFIX})
  set(dllFileName ${CMAKE_SHARED_LIBRARY_PREFIX}${nvName}${fnShared}${CMAKE_SHARED_LIBRARY_SUFFIX})
  string(JOIN "\n" EXT1
    "get_filename_component(PKG_ROOTDIR \${CMAKE_CURRENT_LIST_DIR}/../.. ABSOLUTE)"
    "get_filename_component(PKG_ROOTDIR \${PKG_ROOTDIR} ABSOLUTE) # remove relative parts"
    "if(NOT TARGET ${libName})"
    "  add_library(${libName} STATIC IMPORTED)"
    "  set(libRelease \${PKG_ROOTDIR}/${CMAKE_INSTALL_LIBDIR}/${libFileName})"
    "  if(EXISTS \"\${libRelease}\")"
    "    set_property(TARGET ${libName} APPEND PROPERTY IMPORTED_CONFIGURATIONS RELEASE)"
    "    set_target_properties(${libName} PROPERTIES"
    "      IMPORTED_LINK_INTERFACE_LANGUAGES_RELEASE \"C\""
    "      IMPORTED_LOCATION_RELEASE \"\${libRelease}\""
    "      INTERFACE_INCLUDE_DIRECTORIES \"\${PKG_ROOTDIR}/${CMAKE_INSTALL_INCLUDEDIR}\""
    "      )"
    "  endif()"
    "endif()"
    "set(dllPath \${PKG_ROOTDIR}/${CMAKE_INSTALL_LIBDIR}/${dllFileName})"
    "if(EXISTS \"\${dllPath}\")"
    "  set(NVJPEG2000_DLLS \"\${dllPath}\")"
    "  list(APPEND reqVars NVJPEG2000_DLLS)"
    "endif()"
    ""
    )
endif()
if(COMMAND xpExternPackage)
  xpExternPackage(${nvLibs}
    BASE v0 XPDIFF "bin"
    WEB "https://developer.nvidia.com/nvjpeg"
    DESC "high-performance GPU-accelerated library for decoding JPEG 2000 format images (not available on macOS)"
    LICENSE "[LicenseRef-NVIDIA-SLA](https://docs.nvidia.com/cuda/nvjpeg2000/license.html 'NVIDIA Software License Agreement')"
    )
endif()
if(NOT nvLibs)
  return()
endif()
set(dl https://developer.download.nvidia.com/compute/nvjpeg2000/redist/libnvjpeg_2k)
set(url ${dl}/${os}-${arch}/libnvjpeg_2k-${os}-${arch}-${ver}_cuda${CUDA_VER_MAJOR}-archive.${ext})
set(fcName ${PROJECT_NAME}_fc)
include(FetchContent)
cmake_policy(SET CMP0168 OLD) # don't want to see download progress
FetchContent_Declare(${fcName} URL ${url} URL_HASH SHA256=${sha})
FetchContent_MakeAvailable(${fcName})
string(TOLOWER ${fcName} fcName)
set(pth ${FETCHCONTENT_BASE_DIR}/${fcName}-src)
set(dll ${pth}/${dllPath}/${dllFileName})
if(EXISTS ${dll} AND NOT IS_SYMLINK ${dll})
  set(dlls ${dll})
endif()
file(GLOB hdrs ${pth}/include/${nvName}*.h)
install(FILES ${hdrs} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
install(FILES ${pth}/lib/${libFileName} ${dlls} DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES ${pth}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR})
