cmake_minimum_required(VERSION 3.10)
project(PokerBot VERSION 1.0.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Compiler flags
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang|AppleClang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -O3 -march=native")
elseif(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /O2")
endif()

# Find Intel TBB
find_package(TBB REQUIRED)

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

# Source files
set(SOURCE_FILES
    src/Action.cpp
    src/ActionNode.cpp
    src/BestResponse.cpp
    src/BestResponseTask.cpp
    src/card_utility.cpp
    src/CfrTask.cpp
    src/ChanceNode.cpp
    src/ChanceNodeChild.cpp
    src/GameTree.cpp
    src/Hand.cpp
    src/HandEvaluator.cpp
    src/PlayerState.cpp
    src/RangeManager.cpp
    src/ShowdownTask.cpp
    src/State.cpp
    src/TerminalNode.cpp
    src/Trainer.cpp
    src/TreeBuildSettings.cpp
)

# Header files (for IDE organization)
set(HEADER_FILES
    src/Action.h
    src/ActionNode.h
    src/ActionTypeEnum.h
    src/BestResponse.h
    src/BestResponseTask.h
    src/BetSettings.h
    src/card_utility.h
    src/CfrTask.h
    src/ChanceNode.h
    src/ChanceNodeChild.h
    src/ChanceNodeTypeEnum.h
    src/deck.h
    src/GameTree.h
    src/Hand.h
    src/HandEvaluator.h
    src/Node.h
    src/NodeTypeEnum.h
    src/PlayerState.h
    src/PreflopCombo.h
    src/RangeManager.h
    src/ShowdownTask.h
    src/State.h
    src/StreetEnum.h
    src/TerminalNode.h
    src/TerminalNodeTypeEnum.h
    src/Trainer.h
    src/TreeBuildSettings.h
)

# Create executable
add_executable(PokerBot
    src/program.cpp
    ${SOURCE_FILES}
    ${HEADER_FILES}
)

# Link libraries
target_link_libraries(PokerBot
    TBB::tbb
)

# Set output directory
set_target_properties(PokerBot PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
)

# Installation rules
install(TARGETS PokerBot
    RUNTIME DESTINATION bin
)

install(DIRECTORY data/
    DESTINATION share/pokerbot
    FILES_MATCHING PATTERN "*.dat"
)

# Print build information
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "C++ flags: ${CMAKE_CXX_FLAGS}")
message(STATUS "TBB found: ${TBB_FOUND}")

# Optional: Create documentation target
find_package(Doxygen)
if(DOXYGEN_FOUND)
    set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/docs/Doxyfile.in)
    set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

    if(EXISTS ${DOXYGEN_IN})
        configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
        add_custom_target(doc
            COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            COMMENT "Generating API documentation with Doxygen"
            VERBATIM
        )
    endif()
endif()

# Testing (optional - can be expanded)
enable_testing()

# Print instructions for HandRanks.dat
message(STATUS "")
message(STATUS "===============================================")
message(STATUS "IMPORTANT: Download HandRanks.dat before running!")
message(STATUS "cd data/")
message(STATUS "curl -O https://raw.githubusercontent.com/christophschmalhofer/poker/master/XPokerEval/XPokerEval.TwoPlusTwo/HandRanks.dat")
message(STATUS "===============================================")
message(STATUS "")
