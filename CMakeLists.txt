#  Copyright (c) Microsoft Corporation. All rights reserved.
#  Licensed under the MIT License.

###################################################################################################################

# Select your developer board by removing the # tag from the beginning of the line

# The default board selected is the AVNET Starter Kit.

# If you are NOT using the AVNET board be sure to comment out the AVNET board

set(AVNET TRUE "AVNET Azure Sphere Starter Kit")                
# set(SEEED_STUDIO_RDB TRUE "Seeed Studio Azure Sphere MT3620 Development Kit (aka Reference Design Board or rdb)")
# set(SEEED_STUDIO_MINI TRUE "Seeed Studio Azure Sphere MT3620 Mini Dev Board")

# Uncomment if using Seeed Studio Grove Shield
# set(SEEED_GROVE_SHIELD TRUE "Using Seeed Grove Shield")

###################################################################################################################

cmake_minimum_required (VERSION 3.20)
project (AzureSphereIoTCentral C)

azsphere_configure_tools(TOOLS_REVISION "22.09")
azsphere_configure_api(TARGET_API_SET "14")

add_subdirectory("./LearningPathLibrary")

if(SEEED_GROVE_SHIELD)

add_subdirectory("./MT3620_Grove_Shield_Library")
add_definitions( -DSEEED_GROVE_SHIELD=TRUE )

endif(SEEED_GROVE_SHIELD)

set(Source
    "main.c"
)
source_group("Source" FILES ${Source})

if(AVNET)

    set(Oem
        "./LearningPathLibrary/AVNET/lps22hh_reg.c"
        "./LearningPathLibrary/AVNET/lsm6dso_reg.c"
        "./LearningPathLibrary/AVNET/imu_temp_pressure.c"
        "./LearningPathLibrary/AVNET/light_sensor.c"
        "./LearningPathLibrary/AVNET/board.c"
    )
    source_group("Oem" FILES ${Oem})

    add_definitions( -DOEM_AVNET=TRUE )

endif(AVNET)

if(SEEED_STUDIO_RDB OR SEEED_STUDIO_MINI)

    set(Oem
        "./LearningPathLibrary/SEEED_STUDIO/board.c"
    )
    source_group("Oem" FILES ${Oem})

    add_definitions( -DOEM_SEEED_STUDIO=TRUE )

endif(SEEED_STUDIO_RDB OR SEEED_STUDIO_MINI)

set(ALL_FILES
    ${Source}
    ${Oem}
)

# Create executable
add_executable(${PROJECT_NAME} ${ALL_FILES})
target_compile_definitions(${PROJECT_NAME} PUBLIC AZURE_IOT_HUB_CONFIGURED)
target_link_libraries(${PROJECT_NAME} applibs pthread gcc_s c azsphere_libs)

target_include_directories(${PROJECT_NAME} PUBLIC
                           ./LearningPathLibrary
                           ./ )

if(SEEED_GROVE_SHIELD)
    target_link_libraries(${PROJECT_NAME} MT3620_Grove_Shield_Library)
    target_include_directories(${PROJECT_NAME} PUBLIC ./MT3620_Grove_Shield_Library )
endif(SEEED_GROVE_SHIELD)

target_compile_options(${PROJECT_NAME} PRIVATE -Wno-unknown-pragmas)
set_source_files_properties(./LearningPathLibrary/AVNET/lsm6dso_reg.c PROPERTIES COMPILE_FLAGS -Wno-conversion)
set_source_files_properties(./LearningPathLibrary/AVNET/lps22hh_reg.c PROPERTIES COMPILE_FLAGS -Wno-conversion)

set_source_files_properties(./LearningPathLibrary/AVNET/imu_temp_pressure.c PROPERTIES COMPILE_FLAGS -Wno-float-conversion)
set_source_files_properties(./LearningPathLibrary/AVNET/imu_temp_pressure.c PROPERTIES COMPILE_FLAGS -Wno-incompatible-pointer-types)
set_source_files_properties(./LearningPathLibrary/AVNET/imu_temp_pressure.c PROPERTIES COMPILE_FLAGS -Wno-conversion)


if(AVNET)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "./HardwareDefinitions/avnet_mt3620_sk" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif(AVNET)

if (SEEED_STUDIO_RDB)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "./HardwareDefinitions/mt3620_rdb" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif()

if (SEEED_STUDIO_MINI)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "./HardwareDefinitions/seeed_mt3620_mdb" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif(SEEED_STUDIO_MINI)

azsphere_target_add_image_package(${PROJECT_NAME})
