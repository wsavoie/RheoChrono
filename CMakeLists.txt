cmake_minimum_required(VERSION 2.8)
project(demo_Chrono_Parallel)


#set path for local find modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")
#find_package(ChronoEngine   COMPONENTS unit_IRRLICHT )
OPTION(BUILD_OPENGL "Build the ChronoEngine_OpenGL library" on)
if (${BUILD_OPENGL})
	find_package(ChronoEngine   COMPONENTS unit_OPENGL )
endif()

include(cmake/SBELUtils.cmake)
enable_openmp_support()
enable_cuda_support()

# User specified path to ChronoEngine
SET(CHRONO_INC           ""    CACHE   PATH    "Where is ChronoEngine Source located")

# User specified path to ChronoEngine Libraries
SET(CHRONO_LIB_PATH           ""   CACHE    PATH    "Where is ChronoEngine Lib located")

# Find required Libraries
#FIND_LIBRARY(CHRONO_LIB NAMES ChronoEngine PATHS ${CHRONO_LIB_PATH} REQUIRED)
#FIND_LIBRARY(CHRONO_LIB_IRRLICHT NAMES ChronoEngine_IRRLICHT PATHS ${CHRONO_LIB_PATH} REQUIRED)
FIND_LIBRARY(CHRONO_LIB_PARALLEL NAMES ChronoEngine_Parallel PATHS ${CHRONO_LIB_PATH} REQUIRED)
FIND_LIBRARY(CHRONO_LIB_UTILS NAMES ChronoEngine_ParallelUtils PATHS ${CHRONO_LIB_PATH} REQUIRED)

#SET(LIBRARIES ${CHRONOENGINE_LIBRARIES} ${CHRONO_LIB} ${CHRONO_LIB_PARALLEL} ${CHRONO_LIB_UTILS})
SET(LIBRARIES ${CHRONOENGINE_LIBRARIES} ${CHRONO_LIB_PARALLEL} ${CHRONO_LIB_UTILS})


MESSAGE( STATUS "CHRONOENGINE_INCLUDES:         " ${CHRONOENGINE_INCLUDES} )

SET(CHRONO_PARALLEL_INC  ${CHRONO_INC}/unit_PARALLEL/)
IF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
# ------------------------------------------------------------------------------
# BLAZE -- required for some parallel solvers
# ------------------------------------------------------------------------------
SET(BLAZE_DIR ""  CACHE PATH   "Where is Blaze located?")
SET(BOOST_DIR ""  CACHE PATH   "Where is Boost located?")
set(CHRONO_PARALLEL_INC ${CHRONO_PARALLEL_INC} ${BLAZE_DIR})
set(CHRONO_PARALLEL_INC ${CHRONO_PARALLEL_INC} ${BOOST_DIR})
FIND_PACKAGE(GLEW)
SET(GLEW_DLL  "" CACHE FILEPATH "The GLEW DLL")
SET(GLFW_DLL  "" CACHE FILEPATH "The GLFW DLL")
set(CHRONO_PARALLEL_INC ${CHRONO_PARALLEL_INC} ${GLEW_INCLUDE_DIR})
set(CHRONO_PARALLEL_INC ${CHRONO_PARALLEL_INC} ${GLM_INCLUDE_DIR})
ENDIF()

include_directories(
    ${CHRONOENGINE_INCLUDES}
    ${CHRONO_PARALLEL_INC}
    ${GLFW_INCLUDE_DIR}
    "${CHRONO_LIB_PATH}/../" 
    "${CUDA_TOOLKIT_ROOT_DIR}/include"
    "${CUDA_SDK_ROOT_DIR}/common/inc"
    )

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
SET(DEMOS "")

# Add executables


SET(DEMOS ${DEMOS}
	SmarticlesSystem
	SmarticlesSystem_Dynamic

	#smarticleParallel
#icePar	
)

FOREACH(PROGRAM ${DEMOS})
    MESSAGE(STATUS "...add ${PROGRAM}")

    ADD_EXECUTABLE(${PROGRAM}  "${PROGRAM}.cpp" Smarticle.cpp SmarticleU.cpp CheckPointSmarticles.cpp)
    SOURCE_GROUP(""  FILES "${PROGRAM}.cpp")

    SET_TARGET_PROPERTIES(${PROGRAM} PROPERTIES
        FOLDER test_mixerDVI
        COMPILE_FLAGS "${CH_BUILDFLAGS}"
        LINK_FLAGS "${CH_LINKERFLAG_EXE}"
    )

    TARGET_LINK_LIBRARIES(${PROGRAM} ${LIBRARIES})
    ADD_DEPENDENCIES(${PROGRAM} ${LIBRARIES})

    INSTALL(TARGETS ${PROGRAM} DESTINATION bin)

ENDFOREACH(PROGRAM)


