# Лабораторная работа №3: Автоматизация сборки проектов с помощью CMake

Цель работы: Научиться создавать CMake-скрипты для сборки статических библиотек и приложений, а также управлять зависимостями между ними.

Студент: qNetayS
Дата выполнения: 2026-05-01
Окружение: Debian Linux, GCC, CMake


## 1. Подготовка рабочего окружения

# 1.1 Создание рабочей директории

```shell
mkdir -p ~/qNetayS/my-lab03
cd ~/qNetayS/my-lab03
```
Результат:
/home/vboxuser/qNetayS/my-lab03

## 2.Создание статической библиотеки formatter

# 2.1 Создание файлов библиотеки

```shell
mkdir -p formatter_lib
cd formatter_lib
```

напишем заголовочный файл formatter.h для объявления нужных функций 
```shell
cat > formatter.h << 'EOF'
#pragma once
#include <string>
#include <ostream>

std::string formatter(const std::string& message);
void formatter(std::ostream& os, const std::string& message);
EOF
```
formatter.cpp
```shell
cat > formatter.cpp << 'EOF'
#include "formatter.h"

std::string formatter(const std::string& message) {
    return "[" + message + "]";
}

void formatter(std::ostream& os, const std::string& message) {
    os << "[" << message << "]";
}
EOF
```
Создание CmakeList.txt как инструкцибю для сборки(говорит компилятору откуда брать заголовки и с чем линк)

```shell

cat > CMakeLists.txt << 'EOF'
cmake_minimum_required(VERSION 3.10)
project(formatter)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter STATIC formatter.cpp)
target_include_directories(formatter PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
EOF
``` 
### 2.2 Сборка и проверка результата
Далее сборка и запуск компляции всей структуры
```shell
rm -rf build
cmake -H. -Bbuild
cmake --build build
```

Результат:
-- Configuring done
-- Generating done
[ 50%] Building CXX object CMakeFiles/formatter.dir/formatter.cpp.o
[100%] Linking CXX static library libformatter.a
[100%] Built target formatter

Для большей проверки напишем команду:
```shell
ls -la build/libformatter.a
```

## 3.Содание библиотеки formatter_ex зависимой от formatter)

### 3.1 Создаем папку для нашей библиотеки
```shell
mkdir -p formatter_ex
cd formatter_ex
```
для заголовочного файла будем использовать
стандартный поток из библиотекси с++ string

```shell
cat > formatter_ex.h << 'EOF'
#pragma once
#include <string>
#include <ostream>

std::string formatter_ex(const std::string& message);
void formatter_ex(std::ostream& os, const std::string& message);
EOF
```
далее создаем файл реализации formatter_ex.cpp

```shell
cat > formatter_ex.cpp << 'EOF'
#include "formatter_ex.h"
#include "formatter.h"

std::string formatter_ex(const std::string& message) {
    return formatter(message);
}

void formatter_ex(std::ostream& os, const std::string& message) {
    formatter(os, message);
}
EOF
```
Далее создаем CMakeLists.txt
Самое важное указать что наша новая библиотека зависит от formatter
```shell
cat > CMakeLists.txt << 'EOF'
cmake_minimum_required(VERSION 3.10)
project(formatter_ex)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter_ex STATIC formatter_ex.cpp)

# Говорим, что formatter_ex использует formatter
target_link_libraries(formatter_ex PUBLIC formatter)

# Указываем пути к заголовочным файлам (своим и formatter)
target_include_directories(formatter_ex PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib
)
EOF
```

# 3.2 Сборка библиотеки 
для сборки воспользуемся DCMAKE_PREFIX_PATH для указания
где лежит наша собранная formatter
```shell
rm -rf build
cmake -H. -Bbuild -DCMAKE_PREFIX_PATH=../formatter_lib/build
cmake --build build
```

## 4.Создание прилоложения hello_world

Для начала стоит создать директорию
для нашего нового проекта

```shell
mkdir -p hellow_world
cd hello_world
```
### 4.1 Создадим main.cpp
главный программный файл
в нашем случае просто выводит hello world в квадратных скобках
```shell
cat > main.cpp << 'EOF'
#include "formatter_ex.h"
#include <iostream>

int main() {
    formatter_ex(std::cout, "Hello, World!");
    std::cout << std::endl;
    return 0;
}
EOF
```
### 4.2 Создаем CMakeLists.txt
Хранит указатели где искать и что подключать

```shell
cat > CMakeLists.txt << 'EOF'
cmake_minimum_required(VERSION 3.10)
project(hello_world)

set(CMAKE_CXX_STANDARD 11)
add_executable(hello_world main.cpp)

# Пути к заголовочным файлам
target_include_directories(hello_world PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib
)

# Подключаем обе библиотеки (formatter_ex вызывает formatter)
target_link_libraries(hello_world PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex/build/libformatter_ex.a
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib/build/libformatter.a
)
EOF
```
### 4.3 Сборка
```shell
rm -rf build
cmake -H. -Bbuild
cmake --build build
```
После этого можно затестировать приложение запустив его

```shell
./build/hello_world
```
Результат:[Hello, World!]

## 5.Создание библиоткеи solver_lib для решения квадратных уравнений 
```shell
mkdir -p solver_lib
cd solver_lib
```

### 5.1 Создаем заголовочный файл solver.h
```shell
cat > solver.h << 'EOF'
#pragma once
#include <complex>
#include <utility>

using solution = std::pair<std::complex<double>, std::complex<double>>;
solution solve_quadratic(double a, double b, double c);
EOF
```
### 5.2 Создать общий файл реализации solver.cpp

в ней будет формула дискриминанта и корней уравнения 
```shell
cat > solver.cpp << 'EOF'
#include "solver.h"
#include <cmath>

solution solve_quadratic(double a, double b, double c) {
    double discriminant = b * b - 4 * a * c;
    std::complex<double> sqrt_disc = std::sqrt(std::complex<double>(discriminant, 0));
    return {
        (-b + sqrt_disc) / (2 * a),
        (-b - sqrt_disc) / (2 * a)
    };
}
EOF
```
### 5.3 Создаем CMakeLists.txt 

все как в прошыле разы 
```shell
cat > CMakeLists.txt << 'EOF'
cmake_minimum_required(VERSION 3.10)
project(solver_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(solver_lib STATIC solver.cpp)
target_include_directories(solver_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
EOF
```


### 5.4 итоговая сборка

```shell
rm -rf build
cmake -H. -Bbuild
cmake --build build
```

## 6.Создаем основное приложение solver
для начала нужно создать папку 
```shell
mkdir -p solver
cd solver
```

### 6.1 Создание основного срр файла  main.cpp
для решения x^3 - 3x + 2 = 0 выводим корни 

```shell
cat > main.cpp << 'EOF'
#include "formatter_ex.h"
#include "solver.h"
#include <iostream>
#include <string>

int main() {
    std::cout << "Solving: 1*x^2 - 3*x + 2 = 0" << std::endl;
    
    auto [root1, root2] = solve_quadratic(1, -3, 2);
    
    formatter_ex(std::cout, "Root 1: " + std::to_string(root1.real()));
    std::cout << std::endl;
    formatter_ex(std::cout, "Root 2: " + std::to_string(root2.real()));
    std::cout << std::endl;
    
    return 0;
}
EOF
```

### 6.2 Создание общего файла привязки CMakeLists.txt
Подключаем все три библиотеки 
```shell
cat > CMakeLists.txt << 'EOF'
cmake_minimum_required(VERSION 3.10)
project(solver)

set(CMAKE_CXX_STANDARD 11)
add_executable(solver main.cpp)

# Пути к заголовочным файлам
target_include_directories(solver PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../solver_lib
)

# Линкуем все три библиотеки
target_link_libraries(solver PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex/build/libformatter_ex.a
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib/build/libformatter.a
    ${CMAKE_CURRENT_SOURCE_DIR}/../solver_lib/build/libsolver_lib.a
)
EOF
```
### 6.3 Итоговая сборка и запуск
```shell
rm -rf build
cmake -H. -Bbuild
cmake --build build
```
Запускаем 
```shell
./build/solver
```
Результат:
Solving: 1*x^2 - 3*x + 2 = 0
/[Root 1: 2]
/[Root 2: 1]

##  7.Итговая структура
```shell
~/qNetayS/my-lab03/
├── formatter_lib/          # библиотека для форматирования
│   ├── formatter.h
│   ├── formatter.cpp
│   ├── CMakeLists.txt
│   └── build/libformatter.a
├── formatter_ex/           # расширенная библиотека (зависит от formatter)
│   ├── formatter_ex.h
│   ├── formatter_ex.cpp
│   ├── CMakeLists.txt
│   └── build/libformatter_ex.a
├── hello_world/            # приложение, использующее formatter_ex
│   ├── main.cpp
│   ├── CMakeLists.txt
│   └── build/hello_world
├── solver_lib/             # библиотека для решения уравнений
│   ├── solver.h
│   ├── solver.cpp
│   ├── CMakeLists.txt
│   └── build/libsolver_lib.a
└── solver/                 # приложение, использующее formatter_ex и solver_lib
    ├── main.cpp
    ├── CMakeLists.txt
    └── build/solver
```
