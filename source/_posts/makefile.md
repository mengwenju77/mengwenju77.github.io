---
title: makefile
---

# Makefile

从一个例子开始：

`main.cpp`文件

```c++
#include <iostream>
#include "functions.h"
using namespace std;
 
int main()
{
    printmain();
    cout << "This is main:" << endl;
    cout << "The factorial of 5 is: " << factorial(5) << endl;
    return 0;
}
```

`function.cpp`文件

```c++
#include "functions.h"
 
int functions(int n)
{
    if(n == 1)
            return 1;
    else
            return n * functions(n-1);
}
```

`printhello.cpp`文件

```c++
#include <iostream>
#include "functions.h"
 
using namespace std;
 
void printhello()
{
    int i;
    cout << "Hello world" << endl;
}
```

`function.h`文件

```c++
#ifndef _FUNCTIONS_H_
#define _FUNCTIONS_H_
void printhello();
int function(int n);
#endif
```

目录结构：

```bash
|-- makefilelearn
|   |-- inc
|   |   `-- functions.h
|   |-- main.cpp
|   |-- main.o
|   `-- src
|       |-- functions.cpp
|       `-- printhello.cpp
```

**编译方法1：使用命令分别编译**

```bash
# makefilelear目录下：
g++ main.cpp -c
# makefilelearn/src目录下：
g++ functions.cpp -c
g++ printhello.cpp -c
```

生成3个`.o`文件，目录结构：

```bash
|-- makefilelearn
|   |-- inc
|   |   |-- functions.h
|   |   `-- functions.h.gch
|   |-- main.cpp
|   |-- main.o
|   `-- src
|       |-- functions.cpp
|       |-- functions.o
|       |-- printhello.cpp
|       `-- printhello.o
```

然后链接在一起：

```c++
g++ main.o src/functions.o src/printhello.o -o main
```

生成可执行程序`main`，运行：`./main`

```bash
Hello world
This is main:
The function of 5 is: 120
```

总结：虽然能编译，并且运行，但是文件多了的话，很麻烦

**注意：**

```bash
g++ main.cpp -c   #只编译不链接
g++ main.cpp -o   #编译并链接
```



**编译方法2：makefile--版本1**

目录结构：

```bash
|-- makefilelearn
|   |-- inc
|   |   `-- functions.h
|   |-- main
|   |-- main.cpp
|   |-- Makefile
|   `-- src
|       |-- functions.cpp
|       `-- printhello.cpp

```

```bash
# VERSION 1
# main为生成的可执行文件，依赖于后面的三个.cpp文件
# g++前面加一个TAB的空格
main: main.cpp src/printhello.cpp src/functions.cpp
	g++ -o main main.cpp src/printhello.cpp src/functions.cpp
```

编译好makefile文件之后使用`make`命令进行编译

缺点：全部编译，如果文件多，要编很久

**编译方法3：makefile--版本2**

```bash
# VERSION 2
# 定义变量
CXX = g++
TARGET = main
OBJ = main.o printhello.o functions.o
# make时执行g++ 先找TARGET，TARGET不存在找OBJ，OBJ不存在，编译三个.cpp文件生成.o文件
# 然后再编译OBJ文件，生成可执行文件hello
$(TARGET): $(OBJ)
	$(CXX) -o $(TARGET) $(OBJ)
# main.o这样来的，编译main.cpp生成
main.o: main.cpp
	$(CXX) -c main.cpp
printhello.o: src/printhello.cpp
	$(CXX) -c src/printhello.cpp
functions.o: src/functions.cpp
	$(CXX) -c src/functions.cpp
```

好处就是只编译修改的文件，没修改的文件不编译

**编译方法4：makefile--版本3**

```bash
# VERSION 4
CXX = g++
TARGET = hello
# 所有当前目录的.cpp文件都放在SRC里面
SRC = $(wildcard *.cpp) $(wildcard src/*.cpp)
# 把SRC里面的.cpp文件替换为.o文件
OBJ = $(patsubst %.cpp, %.o,$(SRC))
 
CXXLAGS = -c -Wall
 
$(TARGET): $(OBJ)
	$(CXX) -o $@ $^
 
%.o: %.cpp
	$(CXX) $(CXXLAGS) $< -o $@ 
 
.PHONY: clean
clean:
	rm -f *.o $(TARGET)
```

这样写更通用，添加一个文件后不需要修改makefile文件



> 查看目录结构

安装目录树结构命令(centos7系统)

```bash
yum install -y tree
```

查看目录结构

```c++
查看当第N级目录和文件
tree -L N
比如：查看3级目录
tree -L 3
```

## 总结

**Makefile书写规则：**

- 一个Makefile文件中可以有一个或多个规则

```shell
目标:依赖
	命令(shell 命令)
	
# 目标：最终要生成的文件（伪目标除外）
# 依赖：生成目标所需的文件或是目标
# 命令：通过执行命令对依赖操作生成目标（命令前必须Tab缩进）
```

- Makefile中的其它规则一般都是为第一条规则服务的

**Makefile工作原理：**

- 命令在执行之前，需要先检查规则中的依赖是否存在
  - 如果存在，执行命令
  - 如果不存在，向下检查其它的规则，检查有没有一个规则是用来生成这个依赖的，如果找到了，则执行该规则中的命令
- 检测更新，在执行规则中的命令时，会比较目标和依赖文件的时间
  - 如果依赖的时间比目标的时间晚，需要重新生成目标（先有的依赖再有的目标，如果依赖比目标晚，说明依赖文件被修改了）
  - 如果依赖时间比目标时间早，目标不需要更新，对应规则中的命令不需要被执行。





# Cmake

```bash
cmake_minimum_required(VERSION 2.8)
project(hello)
# C++11
set(CMAKE_CXX_STANDARD 11)
# 添加头文件，如果不想在包含头文件的时候使用相对路径，可以使用在这里添加头文件路径，表示引用的头文件都是在inc目录下的
include_directories(inc)
# 添加源文件，给./src起个别名 DIR_SRCS
aux_source_directory(./src DIR_SRCS)
# 所有需要编译的可执行文件
add_executable(hello ${DIR_SRCS})
```

目录结构

```bash
|-- makefilelearn
|   |-- CMakeLists.txt
|   |-- inc
|   |   `-- functions.h
|   |-- main.cpp
|   `-- src
|       |-- functions.cpp
|       `-- printhello.cpp
```

使用`CMakelists`文件编译

```bash
# 先执行
cmake .
# 再执行
make
```

但会生成很多文件，会比较乱，删起来也比较麻烦。可以创建一个目录`build`，在`build`目录下进行编译，执行：

```bash
# 先执行
cmake ..
# 再执行
make
```

此时，所有的文件都是在`build`目录下生成的，里面有个`hello`可执行文件，想删的话执行把`build`文件夹里的文件删掉就行了











**参考：**

https://www.bilibili.com/video/BV188411L7d2/?spm_id_from=333.337.search-card.all.click&vd_source=79a47c5035e6336414e7ccb0cf2a076d