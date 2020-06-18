project(pwenclave C)

################################################################################
# Source groups
################################################################################
set(no_group_source_files
    "pwenclave_private.pem"
)
source_group("" FILES ${no_group_source_files})

set(Headers
    "pwenclave.h"
    "pwenclave_t.h"
)
source_group("Headers" FILES ${Headers})

set(Sources
    "blockwise.c"
    "chash.c"
    "hmac.c"
    "pbkdf2.c"
    "pwenclave.c"
    "pwenclave_t.c"
    "sha256.c"
)
source_group("Sources" FILES ${Sources})

set(ALL_FILES
    ${no_group_source_files}
    ${Headers}
    ${Sources}
)

################################################################################
# Target
################################################################################
add_library(${PROJECT_NAME} SHARED ${ALL_FILES})

use_props(${PROJECT_NAME} "${CMAKE_CONFIGURATION_TYPES}" "${DEFAULT_CXX_PROPS}")
################################################################################
# Include directories
################################################################################
target_include_directories(${PROJECT_NAME} PUBLIC
    "$ENV{SGXSDKInstallPath}include;"
    "$ENV{SGXSDKInstallPath}include/tlibc;"
    "$ENV{SGXSDKInstallPath}include/stlport"
)

################################################################################
# Compile definitions
################################################################################

################################################################################
# Compile and link options
################################################################################
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:
            /Od;
            /MTd
        >
        $<$<CONFIG:Prerelease>:
            /O2;
            /MT;
            /Gy
        >
        $<$<CONFIG:Release>:
            /O2;
            /MT;
            /Gy
        >
        $<$<CONFIG:Simulation>:
            /Od;
            /MTd
        >
        /W3;
        ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
        ${DEFAULT_CXX_EXCEPTION_HANDLING}
    )
    target_link_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:
            /DEBUG
        >
        $<$<CONFIG:Prerelease>:
            /DEBUG:FULL;
            /OPT:REF;
            /OPT:ICF
        >
        $<$<CONFIG:Release>:
            /DEBUG:FULL;
            /OPT:REF;
            /OPT:ICF
        >
        $<$<CONFIG:Simulation>:
            /DEBUG
        >
        /MANIFEST:NO;
        /INCREMENTAL:NO
    )
endif()

################################################################################
# Post build events
################################################################################
add_custom_command_if(
    TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMANDS
    COMMAND      $<CONFIG:Debug> "$ENV{SGXSDKInstallPath}bin\\win32\\release\\sgx_sign.exe" sign -key "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave_private.pem" -enclave "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.dll" -out "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.signed.dll" -config "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave.config.xml"
    COMMAND $<CONFIG:Prerelease> "$ENV{SGXSDKInstallPath}bin\\win32\\release\\sgx_sign.exe" sign -key "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave_private.pem" -enclave "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.dll" -out "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.signed.dll" -config "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave.config.xml"
    COMMAND    $<CONFIG:Release> "$ENV{SGXSDKInstallPath}bin\\win32\\release\\sgx_sign.exe" gendata -enclave "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.dll" -out "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.hex" -config "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave.config.xml"
    COMMAND $<CONFIG:Simulation> "$ENV{SGXSDKInstallPath}bin\\win32\\release\\sgx_sign.exe" sign -key "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave_private.pem" -enclave "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.dll" -out "$<SHELL_PATH:${OUTPUT_DIRECTORY}>pwenclave.signed.dll" -config "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/>pwenclave.config.xml"
)

################################################################################
# Dependencies
################################################################################
set(ADDITIONAL_LIBRARY_DEPENDENCIES
    "$<$<CONFIG:Debug>:"
        "sgx_trts;"
        "sgx_tservice;"
        "sgx_tcrypto_opt"
    ">"
    "$<$<CONFIG:Prerelease>:"
        "sgx_trts;"
        "sgx_tservice;"
        "sgx_tcrypto_opt"
    ">"
    "$<$<CONFIG:Release>:"
        "sgx_trts;"
        "sgx_tservice;"
        "sgx_tcrypto_opt"
    ">"
    "$<$<CONFIG:Simulation>:"
        "sgx_trts_sim;"
        "sgx_tservice_sim;"
        "sgx_tcrypto"
    ">"
    "sgx_tstdc"
)
target_link_libraries(${PROJECT_NAME} PUBLIC "${ADDITIONAL_LIBRARY_DEPENDENCIES}")

target_link_directories(${PROJECT_NAME} PUBLIC
    "$<$<CONFIG:Debug>:"
        "$ENV{SGXSDKInstallPath}bin/${CMAKE_VS_PLATFORM_NAME}/$<CONFIG>"
    ">"
    "$<$<CONFIG:Prerelease>:"
        "$ENV{SGXSDKInstallPath}bin/${CMAKE_VS_PLATFORM_NAME}/Release"
    ">"
    "$<$<CONFIG:Release>:"
        "$ENV{SGXSDKInstallPath}bin/${CMAKE_VS_PLATFORM_NAME}/$<CONFIG>"
    ">"
    "$<$<CONFIG:Simulation>:"
        "$ENV{SGXSDKInstallPath}bin/${CMAKE_VS_PLATFORM_NAME}/Debug"
    ">"
)

