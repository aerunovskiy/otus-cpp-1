cmake_minimum_required(VERSION 3.12)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VERSION})

option(WITH_GTEST "Whether to build Google test" ON)

include_directories( include )

AUX_SOURCE_DIRECTORY( src SRC )

configure_file(include/version.h.in version.h)

add_executable(${PROJECT_NAME}_cli src/main.cpp)
add_library(${PROJECT_NAME} src/lib.cpp)

set_target_properties(${PROJECT_NAME}_cli ${PROJECT_NAME} PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
)

target_include_directories(${PROJECT_NAME} PRIVATE
        "${CMAKE_BINARY_DIR}"
)

target_link_libraries(${PROJECT_NAME}_cli PRIVATE
        ${PROJECT_NAME}
)

if(WITH_GTEST)
    find_package(GTest REQUIRED)
    add_executable(gtest_version tests/gtest_version.cpp)

    set_target_properties(gtest_version PROPERTIES
            CXX_STANDARD 14
            CXX_STANDARD_REQUIRED ON
    )

    target_link_libraries(gtest_version
            GTest::GTest
            GTest::Main
            ${PROJECT_NAME}
    )
endif()

if(MSVC)
    target_compile_options(${PROJECT_NAME}_cli PRIVATE
            /W4
    )

    target_compile_options(${PROJECT_NAME} PRIVATE
            /W4
    )

    if(WITH_GTEST)
        target_compile_options(gtest_version PRIVATE
                /W4
        )
    endif()
else()
    target_compile_options(${PROJECT_NAME}_cli PRIVATE
            -Wall -Wextra -pedantic -Werror
    )

    target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall -Wextra -pedantic -Werror
    )

    if(WITH_GTEST)
        target_compile_options(gtest_version PRIVATE
                -Wall -Wextra -pedantic -Werror
        )
    endif()
endif()

if(APPLE)
    install(TARGETS ${PROJECT_NAME}_cli RUNTIME DESTINATION .)
    set(CPACK_GENERATOR "DragNDrop")
else()
    install(TARGETS ${PROJECT_NAME}_cli RUNTIME DESTINATION bin)
    set(CPACK_GENERATOR DEB)
endif()

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT alexey.erunovskiy@gmail.com)
include(CPack)

if(WITH_GTEST)
    enable_testing()
    add_test(gtest_version gtest_version)
endif()