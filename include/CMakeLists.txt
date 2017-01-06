cmake_minimum_required(VERSION 3.3)

hunter_add_package(OpenCL)
find_package(OpenCL CONFIG REQUIRED)

project(OpenCL-cpp VERSION 2.0.10)

if(NOT OPENCLPP_VERSION2_ONLY)
	find_package(PythonInterp)

	if(PYTHONINTERP_FOUND)
		add_custom_command(
			OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/CL/cl.hpp
			COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/CL
			COMMAND ${PYTHON_EXECUTABLE} ${CLCPP_SOURCE_DIR}/gen_cl_hpp.py -i ${CLCPP_SOURCE_DIR}/input_cl.hpp -o ${CMAKE_CURRENT_BINARY_DIR}/CL/cl.hpp
			DEPENDS ${CLCPP_SOURCE_DIR}/input_cl.hpp ${CLCPP_SOURCE_DIR}/gen_cl_hpp.py
			COMMENT "Rebuilding cl.hpp ...")
		add_custom_target(generate_clhpp ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/CL/cl.hpp SOURCES ${CLCPP_SOURCE_DIR}/input_cl.hpp)
	else()
		message(FATAL_ERROR "!!!!!!!Unable to locate python, cl.hpp cannot be generated without it. cl2.hpp will still be created")
	endif()
endif()

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/CL/cl2.hpp
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/CL
    COMMAND ${CMAKE_COMMAND} -E copy ${CLCPP_SOURCE_DIR}/input_cl2.hpp ${CMAKE_CURRENT_BINARY_DIR}/CL/cl2.hpp
    DEPENDS ${CLCPP_SOURCE_DIR}/input_cl2.hpp
    COMMENT "Rebuilding cl2.hpp ...")
add_custom_target(generate_cl2hpp ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/CL/cl2.hpp SOURCES ${CLCPP_SOURCE_DIR}/input_cl2.hpp)

add_library(OpenCL-cpp INTERFACE)
target_link_libraries(OpenCL-cpp INTERFACE OpenCL::OpenCL)
	
#Installation
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file("Config.cmake.in" "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS OpenCL-cpp
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/CL
    DESTINATION "include"
    FILES_MATCHING PATTERN "*.hpp"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
