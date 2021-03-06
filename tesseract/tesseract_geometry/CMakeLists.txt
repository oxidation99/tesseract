cmake_minimum_required(VERSION 3.5.0)
project(tesseract_geometry VERSION 0.1.0 LANGUAGES CXX)

find_package(Eigen3 REQUIRED)
find_package(console_bridge REQUIRED)
find_package(tesseract_common REQUIRED)
find_package(octomap REQUIRED)

set(COVERAGE_EXCLUDE /usr/* /opt/* ${CMAKE_CURRENT_LIST_DIR}/test/* /*/gtest/*)

add_code_coverage_all_targets(EXCLUDE ${COVERAGE_EXCLUDE})

find_package(ASSIMP QUIET)
if (NOT ASSIMP_FOUND)
  find_package(PkgConfig REQUIRED)
  # assimp is required, so REQUIRE the second attempt
  pkg_check_modules(ASSIMP REQUIRED assimp)
endif()

# These targets are necessary for 16.04 builds. Remove when Kinetic support is dropped
if(NOT TARGET octomap)
  add_library(octomap INTERFACE IMPORTED)
  set_target_properties(octomap PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${OCTOMAP_INCLUDE_DIRS}")
  set_target_properties(octomap PROPERTIES INTERFACE_LINK_LIBRARIES "${OCTOMAP_LIBRARIES}")
endif()
if(NOT TARGET octomath)
  add_library(octomath INTERFACE IMPORTED)
  set_target_properties(octomath PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${OCTOMAP_INCLUDE_DIRS}")
  set_target_properties(octomath PROPERTIES INTERFACE_LINK_LIBRARIES "${OCTOMAP_LIBRARIES}")
endif()

add_library(${PROJECT_NAME} INTERFACE)
target_link_libraries(${PROJECT_NAME} INTERFACE console_bridge octomap octomath ${ASSIMP_LIBRARIES})
tesseract_target_compile_options(${PROJECT_NAME} INTERFACE)
tesseract_clang_tidy(${PROJECT_NAME})
tesseract_code_coverage(${PROJECT_NAME} ALL EXCLUDE ${COVERAGE_EXCLUDE})
target_include_directories(${PROJECT_NAME} INTERFACE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include>")
target_include_directories(${PROJECT_NAME} SYSTEM INTERFACE
    ${EIGEN3_INCLUDE_DIRS}
    $<TARGET_PROPERTY:tesseract::tesseract_common,INTERFACE_INCLUDE_DIRECTORIES>) #tesseract::tesseract_common Due to bug in catkin, there is an open PR

tesseract_configure_package(${PROJECT_NAME})

# Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}
  DESTINATION include
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

# ADD Examples
add_subdirectory(examples)

if (TESSERACT_ENABLE_TESTING)
  enable_testing()
  tesseract_add_run_tests_target()
  add_subdirectory(test)
endif()
