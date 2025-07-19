cmake_minimum_required(VERSION 3.31)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/.devcontainer/cmake)
include(preproject)
project(nvJPEG2000 VERSION 0.8.1.1)
set(build 40)
set(SHA256_Linux b028f3718f453a71736c01fb8dcbb0174336ea5d69e52fd12a756d6ff5ce785d)
set(SHA256_win64 970308dd3837964455600ce68af2fc0ad5e2b4dc415891e8d255ad1191fc248d)
set(SHA256_aarch64 5d0e61f48dc99c3ac464ceef173c071d8fb6ee0e7e6a83b199291307d9970485)
set(CUDA_VER_MAJOR 12)
include(flags)
include(GNUInstallDirs)
set(XP_INSTALL_CMAKEDIR ${CMAKE_INSTALL_DATADIR}/cmake) # xpPackageDevel
string(TOLOWER ${CMAKE_SYSTEM_NAME} os)
set(ver ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}.${build})
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  set(arch x86_64)
  set(ext zip)
  set(sha ${SHA256_win64})
  set(fnShared _0)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(ext tar.xz)
  set(fnStatic _static)
  if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64|AMD64)$")
    set(arch x86_64)
    set(sha ${SHA256_Linux})
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|ARM64)$")
    set(arch aarch64)
    set(sha ${SHA256_aarch64})
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
endif()
set(url https://developer.download.nvidia.com/compute/nvjpeg2000/redist/libnvjpeg_2k/${os}-${arch}/libnvjpeg_2k-${os}-${arch}-${ver}-archive.${ext})
set(fcName ${PROJECT_NAME}_fc)
include(FetchContent)
cmake_policy(SET CMP0168 OLD) # don't want to see download progress
FetchContent_Declare(${fcName} URL ${url} URL_HASH SHA256=${sha})
FetchContent_MakeAvailable(${fcName})
set(nvName nvjpeg2k)
set(libName xpro::${nvName})
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
xpPackageDevel(LIBRARIES ${libName})
string(TOLOWER ${fcName} fcName)
set(pth ${FETCHCONTENT_BASE_DIR}/${fcName}-src)
set(dll ${pth}/lib/${CUDA_VER_MAJOR}/${dllFileName})
if(EXISTS ${dll} AND NOT IS_SYMLINK ${dll})
  set(dlls ${dll})
endif()
file(GLOB hdrs ${pth}/include/${nvName}*.h)
install(FILES ${hdrs} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
install(FILES ${pth}/lib/${CUDA_VER_MAJOR}/${libFileName} ${dlls} DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES ${pth}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR})
