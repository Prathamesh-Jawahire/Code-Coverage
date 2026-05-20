cmake_minimum_required(VERSION 3.16)

project(OnlineShoppingSystem CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# =========================================
# COVERAGE FLAGS
# =========================================
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} --coverage")

# =========================================
# PROJECT PATHS
# =========================================

set(SHOPPING_SERVER_DIR ${CMAKE_SOURCE_DIR}/online-shopping-system/Server/Headers)

set(SHOPPING_SOURCES
    ${SHOPPING_SERVER_DIR}/Cash.cpp
    ${SHOPPING_SERVER_DIR}/Complaint_Base.cpp
    ${SHOPPING_SERVER_DIR}/Complaint_E.cpp
    ${SHOPPING_SERVER_DIR}/Person.cpp
    ${SHOPPING_SERVER_DIR}/Customer.cpp
    ${SHOPPING_SERVER_DIR}/Employee.cpp
    ${SHOPPING_SERVER_DIR}/Admin.cpp
    ${SHOPPING_SERVER_DIR}/Thread.cpp
    ${SHOPPING_SERVER_DIR}/Server.cpp
)
# Production objects (interactive getch / pauses unchanged)
add_library(shopping_obj_prod OBJECT ${SHOPPING_SOURCES})
target_include_directories(shopping_obj_prod PUBLIC ${SHOPPING_SERVER_DIR})
target_link_libraries(shopping_obj_prod PUBLIC ws2_32)

add_library(shopping_core_prod STATIC $<TARGET_OBJECTS:shopping_obj_prod>)
target_include_directories(shopping_core_prod PUBLIC ${SHOPPING_SERVER_DIR})
target_link_libraries(shopping_core_prod PUBLIC ws2_32)

add_executable(online_shopping_server
    ${CMAKE_SOURCE_DIR}/online-shopping-system/Server/serverProgram.cpp
)
target_include_directories(online_shopping_server PRIVATE
    ${CMAKE_SOURCE_DIR}/online-shopping-system/Server/Headers
)
target_link_libraries(online_shopping_server PRIVATE shopping_core_prod)

# Tests: same sources with non-blocking UI hooks
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    option(BUILD_TESTING "Build and register unit tests" ON)
endif()

include(CTest)
if(BUILD_TESTING)
    add_library(shopping_obj_test OBJECT ${SHOPPING_SOURCES})
    target_include_directories(shopping_obj_test PUBLIC ${SHOPPING_SERVER_DIR})
    target_compile_definitions(shopping_obj_test PRIVATE OSHOP_TEST_BUILD)
    target_link_libraries(shopping_obj_test PUBLIC ws2_32)

    add_library(shopping_core_test STATIC $<TARGET_OBJECTS:shopping_obj_test>)
    target_include_directories(shopping_core_test PUBLIC ${SHOPPING_SERVER_DIR})
    target_link_libraries(shopping_core_test PUBLIC ws2_32)

    enable_testing()
    add_subdirectory(${CMAKE_SOURCE_DIR}/test ${CMAKE_BINARY_DIR}/test)
endif()
