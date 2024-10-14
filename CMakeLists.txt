cmake_minimum_required(VERSION 3.28)

set(VCPKG_TARGET_TRIPLET x64-mingw-dynamic)
set(CMAKE_TOOLCHAIN_FILE "D:/ScoopApp/Scoop/apps/vcpkg/current/scripts/buildsystems/vcpkg.cmake")

project(templates)

find_package(boost COMPONENTS regex system thread hana REQUIRED)

add_executable(${PROJECT_NAME} main.cpp)

target_link_libraries(${PROJECT_NAME} PRIVATE ${Boost_LIBRARIES})
target_include_directories(${PROJECT_NAME} PRIVATE ${Boost_INCLUDE_DIRS})
target_include_directories(${PROJECT_NAME} PRIVATE include)
