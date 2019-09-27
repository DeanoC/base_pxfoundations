cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
get_directory_property(hasParent PARENT_DIRECTORY)
if(NOT hasParent)
	get_filename_component(PARENT_DIR ${CMAKE_CURRENT_SOURCE_DIR} DIRECTORY)
	set_property(GLOBAL PROPERTY GLOBAL_FETCHDEPS_BASE ${PARENT_DIR}/al2o3 )
	set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/out_libs)
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/out_bin)
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/out_bin)
	set(LIB_BASE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/src/libs/)
	include(FetchContent)
	FetchContent_Declare( al2o3 GIT_REPOSITORY https://github.com/DeanoC/al2o3 GIT_TAG master )
	FetchContent_GetProperties(al2o3)
	if(NOT al2o3_POPULATED)
		FetchContent_Populate(al2o3)
		add_subdirectory(${al2o3_SOURCE_DIR} ${al2o3_BINARY_DIR})
	endif()
endif ()

SET_MIN_VERSIONS()
set(LibName base_pxfoundations)
project(${LibName})

set(Interface
		foundation/Ps.hpp
		foundation/PsAlignedMalloc.hpp
		foundation/PsAlloca.hpp
		foundation/PsAllocator.hpp
		foundation/PsAoS.hpp
		foundation/PsArray.hpp
		foundation/PsAtomic.hpp
		foundation/PsBasicTemplates.hpp
		foundation/PsBitUtils.hpp
		foundation/PsBroadcast.hpp
		foundation/PsCpu.hpp
		foundation/PsFoundation.hpp
		foundation/PsFPU.hpp
		foundation/PsHash.hpp
		foundation/PsHashInternals.hpp
		foundation/PsHashMap.hpp
		foundation/PsHashSet.hpp
		foundation/PsInlineAllocator.hpp
		foundation/PsInlineAoS.hpp
		foundation/PsInlineArray.hpp
		foundation/PsIntrinsics.hpp
		foundation/PsMathUtils.hpp
		foundation/PsMutex.hpp
		foundation/PsPool.hpp
		foundation/PsSList.hpp
		foundation/PsSocket.hpp
		foundation/PsSort.hpp
		foundation/PsSortInternals.hpp
		foundation/PsString.hpp
		foundation/PsSync.hpp
		foundation/PsTempAllocator.hpp
		foundation/PsThread.hpp
		foundation/PsTime.hpp
		foundation/PsUserAllocated.hpp
		foundation/PsUtilities.hpp
		foundation/PsVecMath.hpp
		foundation/PsVecMathAoSScalar.hpp
		foundation/PsVecMathAoSScalarInline.hpp
		foundation/PsVecMathSSE.hpp
		foundation/PsVecMathUtilities.hpp
		foundation/PsVecQuat.hpp
		foundation/PsVecTransform.hpp
		foundation/Px.hpp
		foundation/PxAllocatorCallback.hpp
		foundation/PxBitAndData.hpp
		foundation/PxBounds3.hpp
		foundation/PxErrorCallback.hpp
		foundation/PxErrors.hpp
		foundation/PxFlags.hpp
		foundation/PxIntrinsics.hpp
		foundation/PxIO.hpp
		foundation/PxMat33.hpp
		foundation/PxMat44.hpp
		foundation/PxMath.hpp
		foundation/PxMemory.hpp
		foundation/PxPlane.hpp
		foundation/PxPreprocessor.hpp
		foundation/PxProfiler.hpp
		foundation/PxQuat.hpp
		foundation/PxSharedAssert.hpp
		foundation/PxSimpleTypes.hpp
		foundation/PxStrideIterator.hpp
		foundation/PxTransform.hpp
		foundation/PxUnionCast.hpp
		foundation/PxVec2.hpp
		foundation/PxVec3.hpp
		foundation/PxVec4.hpp
		)

set(Src
		PsUtilities.cpp
		PsTempAllocator.cpp
		PsString.cpp
		PsMathUtils.cpp
		PsFoundation.cpp
		PsAssert.cpp
		PsAllocator.cpp
		)

if (WIN32)
	list(APPEND Src windows/PsUWPThread.cpp)
	list(APPEND Src windows/PsWindowsAtomic.cpp)
	list(APPEND Src windows/PsWindowsCpu.cpp)
	list(APPEND Src windows/PsWindowsFPU.cpp)
	list(APPEND Src windows/PsWindowsMutex.cpp)
	list(APPEND Src windows/PsWindowsPrintString.cpp)
	list(APPEND Src windows/PsWindowsSList.cpp)
	list(APPEND Src windows/PsWindowsSocket.cpp)
	list(APPEND Src windows/PsWindowsSync.cpp)
	list(APPEND Src windows/PsWindowsThread.cpp)
endif ()

if (APPLE OR UNIX)
	list(APPEND Src unix/PsUnixAtomic.cpp)
	list(APPEND Src unix/PsUnixCpu.cpp)
	list(APPEND Src unix/PsUnixFPU.cpp)
	list(APPEND Src unix/PsUnixMutex.cpp)
	list(APPEND Src unix/PsUnixPrintString.cpp)
	list(APPEND Src unix/PsUnixSList.cpp)
	list(APPEND Src unix/PsUnixSocket.cpp)
	list(APPEND Src unix/PsUnixSync.cpp)
	list(APPEND Src unix/PsUnixThread.cpp)
endif ()

set(Deps
    al2o3_platform
		)
ADD_LIB(${LibName} "${Interface}" "${Src}" "${Deps}")

set( Tests
		runner.cpp
		)
set( TestDeps
		al2o3_catch2

		utils_simple_logmanager)
ADD_LIB_TESTS(${LibName} "${Interface}" "${Tests}" "${TestDeps}")
target_include_directories(${LibName} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/base_pxfoundations)
target_include_directories(${LibName} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/base_pxfoundations/foundation)

if(WIN32)
	if(${CMAKE_BUILD_TYPE} )
		if (${CMAKE_BUILD_TYPE} STREQUAL "Debug")
			target_compile_definitions(${LibName} PUBLIC _DEBUG)
		else ()
			target_compile_definitions(${LibName} PUBLIC NDEBUG)
		endif ()
	endif()
else()
	target_compile_definitions(${LibName} PUBLIC NDEBUG)
endif()

target_compile_definitions(${LibName} PRIVATE DISABLE_CUDA_PHYSX PX_PHYSX_STATIC_LIB)
