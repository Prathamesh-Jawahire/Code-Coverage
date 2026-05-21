cmake_minimum_required(VERSION 3.16)

# =========================================
# GOOGLE TEST
# =========================================

include(FetchContent)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.14.0
)

set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(googletest)

# =========================================
# COVERAGE
# =========================================

option(ENABLE_COVERAGE "Enable gcov coverage" ON)

# =========================================
# TEST EXECUTABLE
# =========================================

add_executable(unit_tests
    test_complaint.cpp
    test_person_flows.cpp
    test_admin_employee.cpp
    test_thread_smoke.cpp
    test_server_thread.cpp
)

target_include_directories(unit_tests PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

target_link_libraries(unit_tests PRIVATE
    shopping_core_test
    GTest::gtest_main
    ws2_32
)

target_compile_definitions(unit_tests PRIVATE
    OSHOP_TEST_BUILD
)

# =========================================
# COVERAGE FLAGS
# =========================================

if(ENABLE_COVERAGE AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")

    target_compile_options(unit_tests PRIVATE
        --coverage
        -O0
        -g
    )

    target_link_options(unit_tests PRIVATE
        --coverage
    )

endif()

# =========================================
# TEST REGISTRATION
# =========================================

enable_testing()

add_test(
    NAME unit_tests
    COMMAND unit_tests
)

# Optional automatic discovery
include(GoogleTest)

gtest_discover_tests(unit_tests)

# =========================================
# DEBUG OUTPUT
# =========================================

message(STATUS "Coverage Enabled: ${ENABLE_COVERAGE}")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "Registered test executable: unit_tests")