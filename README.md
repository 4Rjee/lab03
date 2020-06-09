## Laboratory work III

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab03** на сервисе **GitHub**
- [x] 2. Ознакомиться со ссылками учебного материала
- [x] 3. Выполнить инструкцию учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial


```sh
$ export GITHUB_USERNAME=4Rjee
```

```sh
$ cd ${GITHUB_USERNAME}/workspace
# пуш в стек директорий
$ pushd .
$ source scripts/activate
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab02.git projects/lab03
$ cd projects/lab03
# меняем удалённые репозиторий
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab03.git
```

```sh
# компиляция файла sources/print.cpp в объектный файл
# -I флаг добавлен для поиска заголовков в ./include
$ g++ -std=c++11 -I./include -c sources/print.cpp
$ ls print.o
# nm - комманда для просмотра таблицы имен бинарного файла
$ nm print.o | grep print
$ ar rvs print.a print.o
# определение типа файла
$ file print.a
$ g++ -std=c++11 -I./include -c examples/example1.cpp
$ ls example1.o
# линковка со статической библиотекой (поскольку есть неопределённые имена) 
$ g++ example1.o print.a -o example1
# будет выполнен переход на новую строку в случае возврата нуля example1
$ ./example1 && echo
```

```sh
$ g++ -std=c++11 -I./include -c examples/example2.cpp
$ nm example2.o
# линковка со статической библиотекой (поскольку есть неопределённые имена) 
$ g++ example2.o print.a -o example2
$ ./example2
$ cat log.txt && echo
```

```sh
$ rm -rf example1.o example2.o print.o
$ rm -rf print.a
$ rm -rf example1 example2
$ rm -rf log.txt
```

```sh
# указываем минимальную требуемую версию CMake и имя проекта
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(print)
EOF
```

```sh
# ставим стандарт языка c++11 и объявляем его необходимым
$ cat >> CMakeLists.txt <<EOF
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
EOF
```

```sh
# добавляем статическую библиотеку print
$ cat >> CMakeLists.txt <<EOF
add_library(print STATIC \${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
EOF
```

```sh
# указываем путь для поиска заголовков для всего проекта
$ cat >> CMakeLists.txt <<EOF
include_directories(\${CMAKE_CURRENT_SOURCE_DIR}/include)
EOF
```

```sh
# помещаем сгенерированные файлы для сборки в _build/
$ cmake -H. -B_build
# cобираем проект
$ cmake --build _build
```

```sh
# добавляем к целям помимо библиотеки два исполняемых файла
$ cat >> CMakeLists.txt <<EOF
add_executable(example1 \${CMAKE_CURRENT_SOURCE_DIR}/examples/example1.cpp)
add_executable(example2 \${CMAKE_CURRENT_SOURCE_DIR}/examples/example2.cpp)
EOF
```

```sh
# линкуем библиотеку с файлами
$ cat >> CMakeLists.txt <<EOF
target_link_libraries(example1 print)
target_link_libraries(example2 print)
EOF
```

```sh
# сборка всего проекта
$ cmake --build _build
# сборка определённых целей (поскольку в cmake реализовано кэширование,
# эти команды после нахождения уже собранных целей отметят факт готовности нужных файлов)
$ cmake --build _build --target print
$ cmake --build _build --target example1
$ cmake --build _build --target example2
```

```sh
# информация о файле
$ ls -la _build/libprint.a
# аналогичный вышеуказанным командам запуск файлов
# только теперь файлы собраны автоматически
$ _build/example1 && echo
hello
$ _build/example2
$ cat log.txt && echo
hello
$ rm -rf log.txt
```

```sh
# перемещаем CMakeLists из репозитория в текущую директорию
$ git clone https://github.com/tp-labs/lab03 tmp
$ mv -f tmp/CMakeLists.txt .
$ rm -rf tmp
```

```sh
$ cat CMakeLists.txt
# -D флаг используется для определения какой-то опции
# в данном случае идёт определение пути установки
$ cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
$ cmake --build _build --target install
$ tree _install
```

```sh
$ git add CMakeLists.txt
$ git commit -m"added CMakeLists.txt"
$ git push origin master
```

## Homework

Представьте, что вы стажер в компании "Formatter Inc.".
### Задание 1
Вам поручили перейти на систему автоматизированной сборки **CMake**.
Исходные файлы находятся в директории [formatter_lib](formatter_lib).
В этой директории находятся файлы для статической библиотеки *formatter*.
Создайте `CMakeList.txt` в директории [formatter_lib](formatter_lib),
с помощью которого можно будет собирать статическую библиотеку *formatter*.

```sh
$ git clone https://github.com/tp-labs/lab03.git formatter
$ cd formatter
$ cat > formatter_lib/CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(formatter_lib)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(formatter STATIC ${CMAKE_CURRENT_LIST_DIR}/formatter.cpp)
include_directories(${CMAKE_CURRENT_LIST_DIR})
EOF
```


### Задание 2
У компании "Formatter Inc." есть перспективная библиотека,
которая является расширением предыдущей библиотеки. Т.к. вы уже овладели
навыком созданием `CMakeList.txt` для статической библиотеки *formatter*, ваш 
руководитель поручает заняться созданием `CMakeList.txt` для библиотеки 
*formatter_ex*, которая в свою очередь использует библиотеку *formatter*.

```sh
$ cat > formatter_ex_lib/CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(formatter_ex_lib)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
include(${CMAKE_CURRENT_LIST_DIR}/../formatter_lib/CMakeLists.txt)
add_library(formatter_ex STATIC ${CMAKE_CURRENT_LIST_DIR}/formatter_ex.cpp)
target_link_libraries(formatter_ex formatter)
include_directories(${CMAKE_CURRENT_LIST_DIR})
EOF
```


### Задание 3
Конечно же ваша компания предоставляет примеры использования своих библиотек.
Чтобы продемонстрировать как работать с библиотекой *formatter_ex*,
вам необходимо создать два `CMakeList.txt` для двух простых приложений:
* *hello_world*, которое использует библиотеку *formatter_ex*;
* *solver*, приложение которое испольует статические библиотеки *formatter_ex* и *solver_lib*.

```sh
$ cat > hello_world_application/CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(hello_world_application)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
include(${CMAKE_CURRENT_LIST_DIR}/../formatter_ex_lib/CMakeLists.txt)
add_executable(hello_world ${CMAKE_CURRENT_LIST_DIR}/hello_world.cpp)
target_link_libraries(hello_world formatter_ex)
EOF
```

```sh
$ cat > solver_application/CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(solver_application)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
include(${CMAKE_CURRENT_LIST_DIR}/../formatter_ex_lib/CMakeLists.txt)
# specify rules for target solver
add_library(libsolver STATIC ${CMAKE_CURRENT_LIST_DIR}/../solver_lib/solver.cpp)
include_directories(${CMAKE_CURRENT_LIST_DIR}/../solver_lib)
add_executable(solver ${CMAKE_CURRENT_LIST_DIR}/equation.cpp)
target_link_libraries(solver formatter_ex libsolver)
EOF
```

    © 2020 GitHub, Inc.
    Terms
    Privacy
    Security
    Status
    Help

    Contact GitHub
    Pricing
    API
    Training
    Blog
    About

