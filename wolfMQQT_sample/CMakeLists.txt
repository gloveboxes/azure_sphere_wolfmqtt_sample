#  Copyright (c) Microsoft Corporation. All rights reserved.
#  Licensed under the MIT License.

###################################################################################################################

# Select your developer board by removing the # tag from the beginning of the line

# The default board selected is the AVNET Azure Sphere Starter Kit Revision 1.

# If you are NOT using the AVNET Revision 1 board be sure to comment out the AVNET board

set(AVNET TRUE "AVNET Azure Sphere Starter Kit Revision 1 ")    
# set(AVNET_REV_2 TRUE "AVNET Azure Sphere Starter Kit Revision 2 ")             
# set(SEEED_STUDIO_RDB TRUE "Seeed Studio Azure Sphere MT3620 Development Kit (aka Reference Design Board or rdb)")
# set(SEEED_STUDIO_MINI TRUE "Seeed Studio Azure Sphere MT3620 Mini Dev Board")

###################################################################################################################

cmake_minimum_required (VERSION 3.10)
project (AzureSphereAzureIoT C)

azsphere_configure_tools(TOOLS_REVISION "21.01")
azsphere_configure_api(TARGET_API_SET "7")

add_compile_definitions(DEFAULT_MQTT_HOST="mosquitto.australiaeast.cloudapp.azure.com")
add_compile_definitions(DEFAULT_MQTT_SECURE_PORT=8884)
add_compile_definitions(MQTT_CA_CERTIFICATE="certs/ca.crt")
add_compile_definitions(MQTT_CLIENT_PRIVATE_KEY="certs/client.key")
add_compile_definitions(MQTT_CLIENT_CERTIFICATE="certs/client.crt")


add_subdirectory("../LearningPathLibrary" out)

add_compile_definitions(WOLFMQTT_MULTITHREAD)
add_compile_definitions(WOLFMQTT_DISCONNECT_CB)
add_compile_definitions(WOLFMQTT_NONBLOCK)
add_compile_definitions(WOLFMQTT_NO_STDIO)
add_compile_definitions(WOLFSSL_USER_SETTINGS)
# add_compile_definitions(WOLFMQTT_V5)
add_compile_definitions(__linux__)
add_compile_definitions(MQTT_USE_WOLFSSL)
add_compile_definitions(ENABLE_MQTT_TLS)
add_compile_definitions(NO_DH)
add_compile_definitions(AZURE_SPHERE)
add_compile_definitions(WOLFMQTT_USER_SETTINGS)





set(Source
    "main.c"
)
source_group("Source" FILES ${Source})


set(wolfMQTT
    "wolfMQTT/src/mqtt_client.c"
    "wolfMQTT/src/mqtt_packet.c"
    "wolfMQTT/src/mqtt_socket.c"

    "multithread.c"
    "wolfMQTT/examples/mqttexample.c"
    "wolfMQTT/examples/mqttnet.c"
)
source_group("wolfMQTT" FILES ${wolfMQTT})

set(ALL_FILES
    ${Source}
    ${Oem}
    ${wolfMQTT}
)

# Create executable
add_executable(${PROJECT_NAME} ${ALL_FILES})

target_include_directories(${PROJECT_NAME} PUBLIC wolfMQTT ./ )

target_compile_definitions(${PROJECT_NAME} PUBLIC AZURE_IOT_HUB_CONFIGURED)
target_link_libraries(${PROJECT_NAME} applibs pthread gcc_s c azsphere_libs wolfssl tlsutils )

target_include_directories(${PROJECT_NAME} PUBLIC ../LearningPathLibrary )

target_compile_options(${PROJECT_NAME} PRIVATE -Wno-unknown-pragmas)

if (SEEED_STUDIO_RDB OR SEEED_STUDIO_MINI)
    target_include_directories(${PROJECT_NAME} PUBLIC 
                            ../LearningPathLibrary/SEEED_STUDIO
                            )
endif (SEEED_STUDIO_RDB OR SEEED_STUDIO_MINI)

if(AVNET OR AVNET_REV_2)

    target_include_directories(${PROJECT_NAME} PUBLIC 
                            ../LearningPathLibrary/AVNET
                            )
endif(AVNET OR AVNET_REV_2)

if(AVNET)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "../HardwareDefinitions/avnet_mt3620_sk" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif(AVNET)

if(AVNET_REV_2)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "../HardwareDefinitions/avnet_mt3620_sk_rev2" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif(AVNET_REV_2)

if (SEEED_STUDIO_RDB)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "../HardwareDefinitions/mt3620_rdb" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif()

if (SEEED_STUDIO_MINI)
    azsphere_target_hardware_definition(${PROJECT_NAME} TARGET_DIRECTORY "../HardwareDefinitions/seeed_mt3620_mdb" TARGET_DEFINITION "azure_sphere_learning_path.json")
endif(SEEED_STUDIO_MINI)

azsphere_target_add_image_package(${PROJECT_NAME} RESOURCE_FILES "certs/ca.crt" "certs/client.key" "certs/client.crt" )