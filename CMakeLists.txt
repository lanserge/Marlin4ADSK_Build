cmake_minimum_required(VERSION 3.0)

set(ArduinoToolchainVersion 1.8.8)
SET(ArduinoBoardString "arduino:sam:arduino_due_x")
#SET(ArduinoBoardString "arduino:avr:mega:cpu=atmega2560")

set(DOWNLOADS ${CMAKE_CURRENT_SOURCE_DIR}/Downloads)

make_directory(${DOWNLOADS})

include(cmake_arduino_local_install/arduino-local-install.cmake)

set(ARDUINO_SDK_PATH ${CMAKE_CURRENT_SOURCE_DIR}/arduino-${ArduinoToolchainVersion})

if(NOT EXISTS ${ARDUINO_SDK_PATH})
install_arduino_localy(${ArduinoToolchainVersion} ${CMAKE_CURRENT_SOURCE_DIR} ${DOWNLOADS})
endif()

set(ARDUINO_EXE ${ARDUINO_SDK_PATH}/arduino)

execute_process(
    COMMAND ${ARDUINO_EXE} --preferences-file preferences.txt --install-boards arduino:sam
    WORKING_DIRECTORY ${ARDUINO_SDK_PATH}
    RESULT_VARIABLE SAM_install_result
    ERROR_VARIABLE SAM_install_error_log
)

if( (NOT SAM_install_result) OR (SAM_install_error_log MATCHES "already installed") )
    message("Arduino support for SAM boards installed")
else()
    message(FATAL_ERROR "Arduino support for SAM boards failed to install")
endif()

function(get_folder_sources var folder)
    set(SRC_FILES ${${var}})
    foreach(pattern IN LISTS ARGN)
        file(GLOB_RECURSE FILES ${folder}/${pattern})
        list(APPEND SRC_FILES ${FILES})
    endforeach()
    set(${var} ${SRC_FILES} PARENT_SCOPE)
endfunction()

project(marlin4adsk_build LANGUAGES C CXX)

add_custom_target(InstallSAM
    COMMAND ${ARDUINO_EXE} --preferences-file preferences.txt --install-boards arduino:sam
    WORKING_DIRECTORY ${ARDUINO_SDK_PATH}
)

get_folder_sources(MARLIN_SRC_FILES Marlin *.h *.c *.cpp)

set(MARLIN_PROJ_FILENAME Marlin.ino)
set(MARLIN_PROJ_FILE ${CMAKE_SOURCE_DIR}/Marlin/Marlin/${MARLIN_PROJ_FILENAME})

add_custom_command(OUTPUT ${MARLIN_PROJ_FILENAME}.bin ${MARLIN_PROJ_FILENAME}.elf
    COMMAND ${ARDUINO_EXE} --verify --verbose --board ${ArduinoBoardString} ${MARLIN_PROJ_FILE} --pref build.path="${CMAKE_BINARY_DIR}"
    DEPENDS ${MARLIN_SRC_FILES}
)

add_custom_target(Marlin
    DEPENDS ${MARLIN_PROJ_FILENAME}.elf
)

#add_executable(marlin4adsk_build main.c)

#install(TARGETS marlin4adsk_build RUNTIME DESTINATION bin)
