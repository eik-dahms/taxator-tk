PROJECT(taxator-tk)
cmake_minimum_required( VERSION 2.6 )

# internal variables
set( boost_version_required 1.34.1 )
set( boost_version_required_enc 103401 )
set( project_name taxator-tk )

# cmake requirements detection
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake-modules ${CMAKE_MODULE_PATH})
# find_package(Sqlite3)
find_package(Threads REQUIRED)  # necessary workaround for cmake boost detection bug (not linking POSIX threads library)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package( Boost "${boost_version_required}" REQUIRED COMPONENTS program_options system filesystem thread )
if( Boost_FOUND )
	if( Boost_VERSION LESS "${boost_version_required_enc}" )
    message( FATAL_ERROR "FOUND INCORRECT BOOST VERSION: ${Boost_LIB_VERSION},\n${project_name} requires at least version ${boost_version_required} ")
  else()
		include_directories(${Boost_INCLUDE_DIRS})
		link_directories(${Boost_LIBRARY_DIRS})
	endif()
else()
  message( FATAL_ERROR "Unable to find a Boost version. Did you set BOOST_ROOT?" )
endif()

include_directories( "includes-external" )
set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -pedantic -Wno-long-long -Wno-variadic-macros -fpermissive -O2 -march=native") #-g for debuggin, -m32 for x32

# apply filtering to alignments file
add_executable( alignments-filter alignments-filter.cpp src/alignmentrecord.cpp src/accessconv.cpp )
target_link_libraries( alignments-filter ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} )

# takes input alignments and predicts a taxon for each query id using various methods and parameters
add_executable( taxator taxator.cpp src/taxontree.cpp src/taxonomyinterface.cpp src/ncbidata.cpp src/accessconv.cpp src/predictionrecord.cpp )
target_link_libraries( taxator ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} ${Boost_THREAD_LIBRARY} ${CMAKE_THREAD_LIBS_INIT} )

# apply filtering to predictions file
add_executable( binner binner.cpp src/taxontree.cpp src/taxonomyinterface.cpp src/ncbidata.cpp src/predictionrecord.cpp src/bioboxes.cpp )
target_link_libraries( binner ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} )

# taxknife 
add_executable( taxknife taxknife.cpp src/taxontree.cpp src/taxonomyinterface.cpp src/ncbidata.cpp )
target_link_libraries( taxknife ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} )

# unittest: constructs the taxonomy from NCBI dump files and tests the structure thoroughly
add_executable( unittest_ncbitaxonomy unittest_ncbitaxonomy.cpp src/ncbidata.cpp src/accessconv.cpp src/taxontree.cpp src/taxonomyinterface.cpp )
target_link_libraries( unittest_ncbitaxonomy ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} )
