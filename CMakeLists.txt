cmake_minimum_required(VERSION 3.5)
project(dts-overlays)

find_program(DTC dtc)
find_program(CPP gcc)
if(NOT DTC OR NOT CPP)
    message(FATAL_ERROR "dtc and gcc are required")
endif()

file(GLOB OVERLAY_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/*.dtso)
file(GLOB_RECURSE OVERLAY_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/*.h ${CMAKE_CURRENT_SOURCE_DIR}/*.dtsi)
list(FILTER OVERLAY_INCLUDES EXCLUDE REGEX "/build/")

add_custom_target(overlays ALL)

foreach(src ${OVERLAY_SOURCES})
    get_filename_component(name ${src} NAME_WE)
    set(preprocessed ${CMAKE_CURRENT_BINARY_DIR}/${name}.dtso.tmp)
    set(output ${CMAKE_CURRENT_BINARY_DIR}/${name}.dtbo)

    # One target per overlay, named after the .dtso file
    add_custom_command(
        OUTPUT ${output}
        COMMAND ${CPP} -E -nostdinc -I./ -undef -D__DTS__ -x assembler-with-cpp -o ${preprocessed} ${src}
        COMMAND ${DTC} -@ -Hepapr -I dts -O dtb -o ${output} ${preprocessed}
        COMMAND ${CMAKE_COMMAND} -E remove ${preprocessed}
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${src} ${OVERLAY_INCLUDES}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        COMMENT "Compiling ${src} => ${name}.dtbo"
        VERBATIM
        )
    add_custom_target(${name} DEPENDS ${output})
    add_dependencies(overlays ${name})
endforeach()
