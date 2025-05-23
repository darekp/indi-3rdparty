cmake_minimum_required(VERSION 3.16)
PROJECT(indi_mi CXX C)

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/")
LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake_modules/")
include(GNUInstallDirs)

set(THREADS_PREFER_PTHREAD_FLAG ON)

find_package(MICAM REQUIRED)
find_package(INDI REQUIRED)
find_package(USB1 REQUIRED)
find_package(Threads REQUIRED)
find_package(CFITSIO REQUIRED)

set(INDI_MI_VERSION_MAJOR 2)
set(INDI_MI_VERSION_MINOR 2)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/indi_miccd.xml.cmake ${CMAKE_CURRENT_BINARY_DIR}/indi_miccd.xml)

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${INDI_INCLUDE_DIR})
include_directories(${CFITSIO_INCLUDE_DIR})
include_directories(${MICAM_INCLUDE_DIR})

########### MI CCD ###########
set(indi_miccd_SRCS
   ${CMAKE_CURRENT_SOURCE_DIR}/mi_ccd.cpp
   )

add_executable(indi_mi_ccd ${indi_miccd_SRCS})
if (APPLE)
    target_link_libraries(indi_mi_ccd ${INDI_LIBRARIES} ${CFITSIO_LIBRARIES} ${MICAM_LIBRARIES} ${USB1_LIBRARIES} Threads::Threads)
else (APPLE)
    target_link_libraries(indi_mi_ccd ${INDI_LIBRARIES} ${CFITSIO_LIBRARIES} ${MICAM_LIBRARIES} ${USB1_LIBRARIES} Threads::Threads rt)
endif(APPLE)

install(TARGETS indi_mi_ccd RUNTIME DESTINATION bin)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/make_mi_ccd_symlink.cmake
"exec_program(\"${CMAKE_COMMAND}\" ARGS -E create_symlink indi_mi_ccd \$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/indi_mi_ccd_usb)\n
 exec_program(\"${CMAKE_COMMAND}\" ARGS -E create_symlink indi_mi_ccd \$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/indi_mi_ccd_eth)\n")

set_target_properties(indi_mi_ccd PROPERTIES POST_INSTALL_SCRIPT ${CMAKE_CURRENT_BINARY_DIR}/make_mi_ccd_symlink.cmake)

########### MI SFW ###########
set(indi_misfw_SRCS
   ${CMAKE_CURRENT_SOURCE_DIR}/mi_sfw.cpp
   )

add_executable(indi_mi_sfw ${indi_misfw_SRCS})
if (APPLE)
    target_link_libraries(indi_mi_sfw ${INDI_LIBRARIES} ${CFITSIO_LIBRARIES} ${MICAM_LIBRARIES} ${USB1_LIBRARIES} Threads::Threads)
else (APPLE)
    target_link_libraries(indi_mi_sfw ${INDI_LIBRARIES} ${CFITSIO_LIBRARIES} ${MICAM_LIBRARIES} ${USB1_LIBRARIES} Threads::Threads rt)
endif(APPLE)

install(TARGETS indi_mi_sfw RUNTIME DESTINATION bin)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/make_mi_sfw_symlink.cmake
"exec_program(\"${CMAKE_COMMAND}\" ARGS -E create_symlink indi_mi_sfw \$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/indi_mi_sfw_usb)\n
 exec_program(\"${CMAKE_COMMAND}\" ARGS -E create_symlink indi_mi_sfw \$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/indi_mi_sfw_eth)\n")

set_target_properties(indi_mi_sfw PROPERTIES POST_INSTALL_SCRIPT ${CMAKE_CURRENT_BINARY_DIR}/make_mi_sfw_symlink.cmake)

##############################

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/indi_miccd.xml DESTINATION ${INDI_DATA_DIR})
