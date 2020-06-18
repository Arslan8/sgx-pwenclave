project(smoketest C)

################################################################################
# Source groups
################################################################################
set(Headers
    "pwenclave_u.h"
)
source_group("Headers" FILES ${Headers})

set(Sources
    "pwenclave_u.c"
    "smoketest.c"
)
source_group("Sources" FILES ${Sources})

set(ALL_FILES
    ${Headers}
    ${Sources}
)

################################################################################
# Target
################################################################################
add_executable(${PROJECT_NAME} ${ALL_FILES})

use_props(${PROJECT_NAME} "${CMAKE_CONFIGURATION_TYPES}" "${DEFAULT_CXX_PROPS}")
set(ROOT_NAMESPACE smoketest)

set_target_properties(${PROJECT_NAME} PROPERTIES
    VS_GLOBAL_KEYWORD "Win32Proj"
)
set_target_properties(${PROJECT_NAME} PROPERTIES
    INTERPROCEDURAL_OPTIMIZATION_PRERELEASE "TRUE"
    INTERPROCEDURAL_OPTIMIZATION_RELEASE    "TRUE"
)
################################################################################
# Include directories
################################################################################
target_include_directories(${PROJECT_NAME} PUBLIC
    "$<$<CONFIG:Debug>:"
        "${CMAKE_CURRENT_SOURCE_DIR}/../pwenclave"
    ">"
    "$<$<CONFIG:Prerelease>:"
        "${CMAKE_CURRENT_SOURCE_DIR}/../pwenclave"
    ">"
    "$ENV{SGXSDKInstallPath}/include"
)

################################################################################
# Compile definitions
################################################################################
target_compile_definitions(${PROJECT_NAME} PRIVATE
    "$<$<CONFIG:Debug>:"
        "_DEBUG"
    ">"
    "$<$<CONFIG:Prerelease>:"
        "EDEBUG;"
        "NDEBUG"
    ">"
    "$<$<CONFIG:Release>:"
        "NDEBUG"
    ">"
    "$<$<CONFIG:Simulation>:"
        "_DEBUG"
    ">"
    "WIN32;"
    "_CONSOLE;"
    "UNICODE;"
    "_UNICODE"
)

################################################################################
# Compile and link options
################################################################################
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:
            /Od;
            ${DEFAULT_CXX_DEBUG_RUNTIME_LIBRARY}
        >
        $<$<CONFIG:Prerelease>:
            /O2;
            /Oi;
            ${DEFAULT_CXX_RUNTIME_LIBRARY};
            /Gy
        >
        $<$<CONFIG:Release>:
            /O2;
            /Oi;
            ${DEFAULT_CXX_RUNTIME_LIBRARY};
            /Gy
        >
        $<$<CONFIG:Simulation>:
            /Od;
            ${DEFAULT_CXX_DEBUG_RUNTIME_LIBRARY}
        >
        /W3;
        ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
        ${DEFAULT_CXX_EXCEPTION_HANDLING};
        /Y-
    )
    string(CONCAT FILE_CL_OPTIONS
        "/Y-"
    )
    source_file_compile_options(pwenclave_u.c ${FILE_CL_OPTIONS})
    target_link_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:
            /INCREMENTAL
        >
        $<$<CONFIG:Prerelease>:
            /OPT:REF;
            /OPT:ICF;
            /INCREMENTAL:NO
        >
        $<$<CONFIG:Release>:
            /OPT:REF;
            /OPT:ICF;
            /INCREMENTAL:NO
        >
        $<$<CONFIG:Simulation>:
            /INCREMENTAL
        >
        /DEBUG;
        /SUBSYSTEM:CONSOLE
    )
endif()

################################################################################
# Dependencies
################################################################################
add_dependencies(${PROJECT_NAME}
    pwenclave
)

set(ADDITIONAL_LIBRARY_DEPENDENCIES
    "$<$<CONFIG:Debug>:"
        "sgx_urts;"
        "sgx_uae_service"
    ">"
    "$<$<CONFIG:Prerelease>:"
        "sgx_urts;"
        "sgx_uae_service"
    ">"
    "$<$<CONFIG:Release>:"
        "sgx_urts;"
        "sgx_uae_service"
    ">"
    "$<$<CONFIG:Simulation>:"
        "sgx_urts_sim;"
        "sgx_uae_service_sim"
    ">"
)
target_link_libraries(${PROJECT_NAME} PUBLIC "${ADDITIONAL_LIBRARY_DEPENDENCIES}")

target_link_directories(${PROJECT_NAME} PUBLIC
    "$<$<CONFIG:Debug>:"
        "$ENV{SGXSDKInstallPath}/bin/${CMAKE_VS_PLATFORM_NAME}/Debug"
    ">"
    "$<$<CONFIG:Prerelease>:"
        "$ENV{SGXSDKInstallPath}/bin/${CMAKE_VS_PLATFORM_NAME}/Release"
    ">"
    "$<$<CONFIG:Release>:"
        "$ENV{SGXSDKInstallPath}/bin/${CMAKE_VS_PLATFORM_NAME}/Release"
    ">"
    "$<$<CONFIG:Simulation>:"
        "$ENV{SGXSDKInstallPath}/bin/${CMAKE_VS_PLATFORM_NAME}/Debug"
    ">"
)

