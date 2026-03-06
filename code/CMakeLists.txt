cmake_minimum_required(VERSION 3.10)
project(liblac C)

# Set C standard
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Add compiler flags for AES-NI support
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -maes -msse2")

# Define source directory
set(SRC_DIR "${CMAKE_SOURCE_DIR}/src")

# Find all .c files in src directory
file(GLOB LAC_SOURCES "${SRC_DIR}/*.c")

if(NOT LAC_SOURCES)
  message(WARNING "No .c files found in ${SRC_DIR}")
endif()

# Create a static library for each .c file
foreach(source_file ${LAC_SOURCES})
  get_filename_component(lib_name ${source_file} NAME_WE)
  
  # Check if corresponding header exists in /include
  if(EXISTS "${CMAKE_SOURCE_DIR}/include/${lib_name}.h")
    add_library(${lib_name} STATIC ${source_file})
    
    # Add include directories (main include and arith subdirectories)
    target_include_directories(${lib_name} PUBLIC 
      "${CMAKE_SOURCE_DIR}/include"
      "${CMAKE_SOURCE_DIR}/include/arith"
      "${CMAKE_SOURCE_DIR}/include/arith/arith_z"
      "${CMAKE_SOURCE_DIR}/include/arith/arith_q"
      "${CMAKE_SOURCE_DIR}/include/arith/arith_qiss"
      "${CMAKE_SOURCE_DIR}/include/arith/arith_qshow"
      "${CMAKE_SOURCE_DIR}/include/arith/arith_real"
    )
    
    # Set output directory for each library
    set_target_properties(${lib_name} PROPERTIES
      ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    )
    
    # Install library
    install(TARGETS ${lib_name}
      ARCHIVE DESTINATION lib
    )
  else()
    message(WARNING "No header file found for ${lib_name}.c")
  endif()
endforeach()

# Install all header files from include directory
file(GLOB HEADER_FILES "${CMAKE_SOURCE_DIR}/include/*.h")
if(HEADER_FILES)
    install(FILES ${HEADER_FILES} DESTINATION include)
endif()