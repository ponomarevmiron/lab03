# Laboratory work III

Данная лабораторная работа посвящена изучению систем автоматизации сборки проекта на примере CMake

- [x] Создать публичный репозиторий с названием lab03 на сервисе GitHub
- [x] Ознакомиться со ссылками учебного материала
- [x] Выполнить инструкцию учебного материала
- [x] Составить отчет и отправить ссылку личным сообщением в Slack

## Tutorial

### Настройка окружения и клонирование

$ export GITHUB_USERNAME=ponomarevmiron
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate  # файл отсутствовал, шаг пропущен
$ git clone https://github.com/${GITHUB_USERNAME}/lab02.git projects/lab03
$ cd projects/lab03
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab03.git

### Создание CMakeLists.txt

Исходные файлы в lab02 отсутствовали, поэтому созданы директории:
$ mkdir -p sources include examples

Создан print.h и print.cpp.

$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(print)
EOF
$ cat >> CMakeLists.txt <<EOF
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
EOF
$ cat >> CMakeLists.txt <<EOF
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
EOF
$ cat >> CMakeLists.txt <<EOF
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
EOF

Проблема: cmake -H. -B_build выдал ошибку совместимости версий.
Решение: sed -i 's/VERSION 3.4/VERSION 3.5/' CMakeLists.txt

$ cmake -H. -B_build
-- Configuring done
-- Build files have been written
$ cmake --build _build
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print

### Добавление example1 и example2

Созданы example1.cpp и example2.cpp.

$ cat >> CMakeLists.txt <<EOF

add_executable(example1 ${CMAKE_CURRENT_SOURCE_DIR}/examples/example1.cpp)
add_executable(example2 ${CMAKE_CURRENT_SOURCE_DIR}/examples/example2.cpp)
EOF
$ cat >> CMakeLists.txt <<EOF

target_link_libraries(example1 print)
target_link_libraries(example2 print)
EOF

$ cmake --build _build
[ 33%] Built target print
[ 66%] Linking CXX executable example1
[100%] Linking CXX executable example2
[100%] Built target example2

$ ls -la _build/libprint.a
-rwxrwxrwx 1 miron miron 2222 Jun  7 00:42 _build/libprint.a

$ _build/example1 && echo
hello

$ _build/example2
$ cat log.txt && echo
hello

$ rm -rf log.txt

### Замена на эталонный CMakeLists.txt

$ git clone https://github.com/tp-labs/lab03 tmp
$ mv -f tmp/CMakeLists.txt .
$ rm -rf tmp
$ cat CMakeLists.txt
cmake_minimum_required(VERSION 3.4)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
option(BUILD_EXAMPLES "Build examples" OFF)
project(print)
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME} RUNTIME DESTINATION bin)
  endforeach()
endif()
install(TARGETS print EXPORT print-config ARCHIVE DESTINATION lib LIBRARY DESTINATION lib)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)

### Сборка с установкой

$ cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
-- Configuring done
-- Build files have been written

$ cmake --build _build --target install
[100%] Built target print
-- Installing: _install/lib/libprint.a
-- Installing: _install/include/print.h
-- Installing: _install/cmake/print-config.cmake
-- Installing: _install/cmake/print-config-noconfig.cmake

$ ls -R _install
_install:
cmake  include  lib
_install/cmake:
print-config-noconfig.cmake  print-config.cmake
_install/include:
print.h
_install/lib:
libprint.a

### Публикация

$ git add CMakeLists.txt
$ git commit -m "added CMakeLists.txt"
$ git push origin main

## Report

$ popd
$ export LAB_NUMBER=03
$ git clone https://github.com/tp-labs/lab03 tasks/lab03
$ mkdir reports/lab03
$ cp tasks/lab03/README.md reports/lab03/REPORT.md
$ cd reports/lab03
$ edit REPORT.md
$ gist REPORT.md

## Homework

Представьте, что вы стажер в компании "Formatter Inc.".

### Задание 1

Вам поручили перейти на систему автоматизированной сборки CMake.
Исходные файлы находятся в директории formatter_lib.
Создан CMakeList.txt для статической библиотеки formatter.

formatter_lib/CMakeLists.txt:
cmake_minimum_required(VERSION 3.5)
project(formatter)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(formatter STATIC formatter.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

Вывод сборки:
[ 10%] Building CXX object formatter_lib/CMakeFiles/formatter.dir/formatter.cpp.o
[ 20%] Linking CXX static library libformatter.a
[ 20%] Built target formatter

### Задание 2

Создан CMakeList.txt для библиотеки formatter_ex,
которая использует библиотеку formatter.

formatter_ex_lib/CMakeLists.txt:
cmake_minimum_required(VERSION 3.5)
project(formatter_ex)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(formatter_ex STATIC formatter_ex.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR} ../formatter_lib)
target_link_libraries(formatter_ex formatter)

Вывод сборки:
[ 30%] Building CXX object formatter_ex_lib/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
[ 40%] Linking CXX static library libformatter_ex.a
[ 40%] Built target formatter_ex

### Задание 3

Созданы CMakeList.txt для приложений hello_world и solver.

hello_world/CMakeLists.txt:
cmake_minimum_required(VERSION 3.5)
project(hello_world)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_executable(hello_world hello_world.cpp)
include_directories(../formatter_ex_lib ../formatter_lib)
target_link_libraries(hello_world formatter_ex formatter)

Вывод сборки hello_world:
[ 70%] Building CXX object hello_world_application/CMakeFiles/hello_world.dir/hello_world.cpp.o
[ 80%] Linking CXX executable hello_world
[ 80%] Built target hello_world

solver_lib/CMakeLists.txt:
cmake_minimum_required(VERSION 3.5)
project(solver_lib)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(solver_lib STATIC solver.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

solver/CMakeLists.txt:
cmake_minimum_required(VERSION 3.5)
project(solver)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_executable(solver solver.cpp)
include_directories(../solver_lib ../formatter_ex_lib ../formatter_lib)
target_link_libraries(solver solver_lib formatter_ex formatter)

Вывод сборки solver_lib:
[ 50%] Building CXX object solver_lib/CMakeFiles/solver_lib.dir/solver.cpp.o
[ 60%] Linking CXX static library libsolver_lib.a
[ 60%] Built target solver_lib

Вывод сборки solver:
[ 90%] Building CXX object solver_application/CMakeFiles/solver.dir/solver.cpp.o
[100%] Linking CXX executable solver
[100%] Built target solver

Проверка всех CMakeLists.txt:
$ find . -name "CMakeLists.txt" | sort
./formatter_ex_lib/CMakeLists.txt
./formatter_lib/CMakeLists.txt
./hello_world/CMakeLists.txt
./solver/CMakeLists.txt
./solver_lib/CMakeLists.txt

## Links

Основы сборки проектов на С/C++ при помощи CMake
CMake Tutorial
C++ Tutorial - make & CMake
Autotools
CMake
