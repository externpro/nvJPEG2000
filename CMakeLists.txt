cmake_minimum_required(VERSION 3.31.5)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/.devcontainer/cmake)
include(preproject)
project(nvJPEG2000 VERSION 0.8.0.1)
set(build 38)
set(SHA256_Linux f0b4f0ab9e19579c547e96ce189c6a93731f662898751a88d62e3caa59472367)
set(SHA256_win64 feabb2f4108805b251f44b0561b194a24041d01207b272a3aa42a664e938e5d4)
set(CUDA_VER_MAJOR 12)
set(webpro_REV NONE)
include(flags)
include(GNUInstallDirs)
set(XP_INSTALL_CMAKEDIR ${CMAKE_INSTALL_DATADIR}/cmake) # xpPackageDevel
string(TOLOWER ${CMAKE_SYSTEM_NAME} os)
set(ver ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}.${build})
set(url https://developer.download.nvidia.com/compute/nvjpeg2000/redist/libnvjpeg_2k/${os}-x86_64/libnvjpeg_2k-${os}-x86_64-${ver}-archive)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  set(url ${url}.zip)
  set(sha ${SHA256_win64})
  set(fnShared _0)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(url ${url}.tar.xz)
  set(sha ${SHA256_Linux})
  set(fnStatic _static)
endif()
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
file(GLOB hdrs ${pth}/include/${nvName}*.h)
file(GLOB dlls ${pth}/lib/${CUDA_VER_MAJOR}/${dllFileName})
install(FILES ${hdrs} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
install(FILES ${pth}/lib/${CUDA_VER_MAJOR}/${libFileName} ${dlls} DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES ${pth}/LICENSE DESTINATION ${CMAKE_INSTALL_DATADIR})
