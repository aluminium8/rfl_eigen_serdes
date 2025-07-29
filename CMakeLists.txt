cmake_minimum_required(VERSION 3.20)

option(BUILD_TESTS "Build the testing suite." OFF)

if(BUILD_TESTS)
    list(APPEND VCPKG_MANIFEST_FEATURES "testing")
endif()

project(rfl_eigen_serdes C CXX)

message(${CMAKE_SOURCE_DIR})

# CTestを有効化（プロジェクトで一度だけ呼び出す）
enable_testing()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /utf-8") # MSVCの場合

message("CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
message("TOOLCHAIN_FILE: ${CMAKE_TOOLCHAIN_FILE}")

if(WIN32)
    message("this is windows")
elseif(UNIX)
    message("this is UNIX-like")
else()
    message("what is this OS...")
endif()

find_package(reflectcpp CONFIG REQUIRED)
find_package(Eigen3 CONFIG REQUIRED)

add_executable(sample ./sample/sample.cpp)
target_include_directories(sample PRIVATE ./include)
target_link_libraries(sample PRIVATE reflectcpp::reflectcpp Eigen3::Eigen)

# BUILD_TESTSがONの場合のみ、以下の処理を実行
if(BUILD_TESTS)
    find_package(boost_unit_test_framework  CONFIG REQUIRED)

    add_executable(serdes_test
        test/test_serdes.cpp
    )

    target_include_directories(serdes_test PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        #${Boost_INCLUDE_DIR}
    )
    target_compile_definitions(serdes_test PRIVATE NOMINMAX)
    target_link_libraries(serdes_test PRIVATE
        Eigen3::Eigen
        reflectcpp::reflectcpp
    #    Boost::unit_test_framework
    )

    add_test(NAME EigenSerializationTests COMMAND $<TARGET_FILE:serdes_test>  )

endif()