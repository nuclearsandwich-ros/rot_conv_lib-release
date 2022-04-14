# Rotations Conversion Library project
cmake_minimum_required(VERSION 3.3)
project(rot_conv)

# Enable C++11
message(STATUS "Enabling C++11")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
add_compile_options(-Wall -Wextra -Wpedantic)

# Include directories
include_directories(${PROJECT_SOURCE_DIR}/include)

#
# Build libraries
#

# Build a static library
add_library(rotconvstatic STATIC
	src/rot_conv.cpp
	src/rot_conv_extras.cpp
)

# Build a shared library
add_library(rotconvshared SHARED
	src/rot_conv.cpp
	src/rot_conv_extras.cpp
)

#
# Build sample executables
#

# Build the sample target directly with the source files
add_executable(rot_conv_sample_direct
	test/rot_conv_sample.cpp
	src/rot_conv.cpp
	src/rot_conv_extras.cpp
)

# Build the sample target with the static library
add_executable(rot_conv_sample_static
	test/rot_conv_sample.cpp
)
target_link_libraries(rot_conv_sample_static
	rotconvstatic
)

# Build the sample target with the shared library
add_executable(rot_conv_sample_shared
	test/rot_conv_sample.cpp
)
target_link_libraries(rot_conv_sample_shared
	rotconvshared
)

#
# Build unit tests
#

# To enable the building of the unit tests do (in the project source directory):
#   git clone https://github.com/google/googletest googletest
# Then re-run cmake and perform a make and run test_rot_conv.

# Build the unit tests if the google test framework is locally found
if(IS_DIRECTORY ${PROJECT_SOURCE_DIR}/googletest)

	# Import google test directory
	add_subdirectory(googletest)
	enable_testing()
	include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

	# Build and add the required unit test
	add_executable(test_rot_conv
		test/test_rot_conv.cpp
	)
	target_link_libraries(test_rot_conv
		rotconvstatic
		gtest_main
		gtest
	)
	add_test(test_rot_conv test_rot_conv)

endif()

# Find ament_cmake
find_package(ament_cmake REQUIRED)

# Find Eigen3 (See https://github.com/ros2/eigen3_cmake_module/blob/master/README.md for why we need eigen3_cmake_module)
find_package(eigen3_cmake_module REQUIRED)
find_package(Eigen3 REQUIRED)

# Install include directory
install(DIRECTORY include/rot_conv/
        DESTINATION include/${PROJECT_NAME})

# Install shared library
install(TARGETS rotconvshared
        EXPORT export_${PROJECT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)

# Link dependencies
ament_target_dependencies(rotconvstatic Eigen3)
ament_target_dependencies(rotconvshared Eigen3)
ament_target_dependencies(rot_conv_sample_direct Eigen3)

ament_export_include_directories(include)
ament_export_libraries(rotconvshared)
ament_export_dependencies(eigen3_cmake_module Eigen3)
ament_package()

# EOF
