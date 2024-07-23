# - CMake Message Colors
if (NOT WIN32)
    string(ASCII 27 Esc)
    set(TAB           "   ")
    set(RESET         "${Esc}[m")
    set(GREEN         "${Esc}[38;2;0;245;0m")
    set(RED           "${Esc}[38;2;255;0;0m")
    set(BLUE          "${Esc}[38;2;113;111;148m")
    set(LIGHT         "${Esc}[38;2;140;145;159m")
    set(ASTRO         "${Esc}[38;2;177;173;201m")
    set(ASTRO2         "${Esc}[38;2;177;173;255m")
    set(TEXT          "${BLUE}")
    set(QAPB          "${ASTRO}[QAPB]${TEXT}")
    set(SUCCESS       "${QAPB} ${GREEN}[SUCCESS]${TEXT}")
    set(FAILED        "${QAPB} ${RED}[FAILED]${TEXT}")
    # set(LOG            "${TAB}${LIGHT}")
endif ()

# - Specify CMake Version
cmake_minimum_required(VERSION 3.10)

# - Project Name
message("${QAPB} Initializing QSSL Project ...${RESET}")
project(run)
message("${QAPB} Set Project Name to ${LIGHT}${PROJECT_NAME}${RESET}")

# - Check Operating System
message("${QAPB} Operating System Name : ${LIGHT}${CMAKE_SYSTEM_NAME}${RESET}")
if (CMAKE_SYSTEM_NAME STREQUAL "Linux" OR CMAKE_SYSTEM_NAME STREQUAL "Darwin" OR CMAKE_SYSTEM_NAME STREQUAL "Android")
    message("${SUCCESS} Validate Operating System${RESET}")
else ()
    message("${FAILED} Invalid Operating System${RESET}")
    return()
endif ()
message("${QAPB} Operating System Version : ${LIGHT}${CMAKE_SYSTEM_VERSION}${RESET}")
message("${QAPB} Operating System Processor: ${LIGHT}${CMAKE_SYSTEM_PROCESSOR}${RESET}")

# - Show Processor Model
execute_process(
    COMMAND bash -c "lscpu | grep -F 'Model name:' | awk '{for (i=3; i<=NF; i++) printf \"%s \", $i; print \"\"}'"
    OUTPUT_VARIABLE CPU_MODEL
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" "" CPU_MODEL "${CPU_MODEL}")
message("${QAPB} Processor Model Name : ${LIGHT}${CPU_MODEL}${RESET}")

# - Show Processor Cores
include(ProcessorCount)
ProcessorCount(NPROC)
message("${QAPB} Processor Cores : ${LIGHT}${NPROC}${RESET}")

# - Show CPU Frequency
execute_process(
    COMMAND bash -c "echo \"$(lscpu | grep -F 'CPU max MHz:' | awk '{print $4}') GHz\""
    OUTPUT_VARIABLE MAX_PROC_SPEED
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" "" MAX_PROC_SPEED "${MAX_PROC_SPEED}")
execute_process(
    COMMAND bash -c "echo \"$(lscpu | grep -F 'CPU min MHz:' | awk '{print $4}') GHz\""
    OUTPUT_VARIABLE MIN_PROC_SPEED
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" "" MIN_PROC_SPEED "${MIN_PROC_SPEED}")
execute_process(
    COMMAND bash -c "cat /proc/cpuinfo | grep 'cpu MHz' | awk '{total += $4; count++} END {print total/count}' | bc -l"
    OUTPUT_VARIABLE CURRENT_PROC_SPEED
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" " GHz" CURRENT_PROC_SPEED "${CURRENT_PROC_SPEED}")
message("${QAPB} Processor Frequency : ${RESET}")
message("${TAB}${TEXT}Maximum ${LIGHT}${MAX_PROC_SPEED}${RESET}")
message("${TAB}${TEXT}Minimum ${LIGHT}${MIN_PROC_SPEED}${RESET}")
message("${TAB}${TEXT}Current ${LIGHT}${CURRENT_PROC_SPEED}${RESET}")

# - Show Cache Line Size
execute_process(
    COMMAND bash -c "lscpu | grep -E 'L1d cache|cache line size' | awk '{print $3}'"
    OUTPUT_VARIABLE CACHE_LINE_SIZE
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" "" CACHE_LINE_SIZE "${CACHE_LINE_SIZE}")
message("${QAPB} Cache Line Size : ${LIGHT}${CACHE_LINE_SIZE}${RESET}")

# - Show Used Storage
execute_process(
    COMMAND bash -c "df -h | awk '/\\/$/ {print $3\" / \"$2\" Used\"}'"
    OUTPUT_VARIABLE DISK_USAGE
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" "" DISK_USAGE "${DISK_USAGE}")
execute_process(
    COMMAND bash -c "echo \"($(df -h | awk '/\\/$/ {printf \"%0.3f%\", ($3/$2) * 100}'))\""
    OUTPUT_VARIABLE DISK_USAGE_PERCENT
    RESULT_VARIABLE RESULT
)
string(REGEX REPLACE "\n" "" DISK_USAGE_PERCENT "${DISK_USAGE_PERCENT}")
message("${QAPB} Used Storage : ${LIGHT}${DISK_USAGE} ${DISK_USAGE_PERCENT}${RESET}")

# - Set C++ Standard
set(CMAKE_CXX_STANDARD 11)

# - Find Required Packages
if (DEFINED _USE_PACKAGES)
    foreach(UserPackage ${_USE_PACKAGES})
        message("${QAPB} Finding Package ${LIGHT}${UserPackage}${TEXT} ...${RESET}")
        find_package(${UserPackage} REQUIRED)
        message("${SUCCESS} Package ${LIGHT}${UserPackage}${TEXT} has been Found${RESET}")
    endforeach()
else ()
    message("${QAPB} No Additional Package Given${RESET}")
endif ()

# - Add Source Files
if (DEFINED _SRC)
    set(DIRECTORY_SET)
    set(HEADER_MACROS)
    foreach(SOURCE_FILE ${_SRC})
        # - Check Source File
        if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${SOURCE_FILE}")
            message("${SUCCESS} Configured File ${LIGHT}${SOURCE_FILE}${RESET}")
            get_filename_component(DIR "${CMAKE_CURRENT_SOURCE_DIR}/${SOURCE_FILE}" DIRECTORY)
            list(APPEND DIRECTORY_SET ${DIR})
            # - Process Header Files
            if("${SOURCE_FILE}" MATCHES "\\.hpp$")
                get_filename_component(BASENAME "${SOURCE_FILE}" NAME_WE)
                string(TOUPPER "${BASENAME}" MACRO_NAME)
                string(REPLACE " " "_" MACRO_NAME "${MACRO_NAME}")
                list(APPEND HEADER_MACROS "HAS_MODULE_${MACRO_NAME}")
            endif()
        else ()
            message("${FAILED} Source File ${LIGHT}${SOURCE_FILE}${TEXT} Not Found${RESET}")
            return()
        endif ()
    endforeach()
    list(REMOVE_DUPLICATES DIRECTORY_SET)
    # - Include Directories
    foreach(DIR ${DIRECTORY_SET})
        message("${SUCCESS} Including Directory ${LIGHT}${DIR}${RESET}")
        include_directories(${DIR})
    endforeach()
else ()
    message("${FAILED} No Selected Source File ...${RESET}")
    return()
endif ()

# - Add Executable
add_executable(${PROJECT_NAME} ${_SRC})

# - Define Project Modules Macros
foreach(MACRO ${HEADER_MACROS})
    target_compile_definitions(${PROJECT_NAME} PRIVATE ${MACRO})
    message("${SUCCESS} Defining Macro ${ASTRO2}${MACRO}${RESET}")
endforeach()

# - Link the Libraries in _USE_PACKAGES
if (DEFINED _USE_PACKAGES)
    message("${QAPB} Linking Packages ...${RESET}")
    foreach(UserPackage ${_USE_PACKAGES})
        if(${UserPackage}_LIBS)
            target_link_libraries(${PROJECT_NAME} PRIVATE ${${UserPackage}_LIBS})
        elseif(${UserPackage}_LIBRARIES)
            target_link_libraries(${PROJECT_NAME} PRIVATE ${${UserPackage}_LIBRARIES})
        else()
            message(FATAL_ERROR "Library variable not found for ${UserPackage}")
        endif()
    endforeach()
endif ()

# - Define QAPB Macro
target_compile_definitions(${PROJECT_NAME} PRIVATE QAPB="Project Compiles Under QAPB Support")
