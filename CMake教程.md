- 介绍

  CMake教程提供了一个循序渐进的指南，涵盖了CMake帮助解决的常见构建系统问题。了解不同的主题如何在示例项目中一起工作是非常有帮助的。

- 步骤

  教程源代码示例可在此归档文件中获得。每个步骤都有自己的子目录，其中包含可以用作起点的代码。本教程示例是渐进式的，因此每一步都为上一步提供了完整的解决方案。

  [Step 1：一个基本的起点](#1)
  [Step 2：添加Library](#2)
  [Step 3：增加库的使用要求](#3)
  [Step 4：安装和测试](#4)
  [Step 5：添加系统自检](#5)
  [Step 6：添加自定义命令和生成文件](#6)
  [Step 7：包装一个安装程序](#7)
  [Step 8：添加对测试仪表板的支持](#8)
  [Step 9：选择静态库或共享库](#9)
  [Step 10：添加生成器表达式](#10)
  [Step 11：增加导出配置](#11)
  [Step 12：打包、调试和发布](#12)
  
  1. <span id = "1">Step 1：一个基本的起点</span>
  
     ​	最基本的项目是由源代码文件构建的可执行文件。对于简单的项目，只需要一个三行的CMakeLists.txt文件。		这将是我们教程的起点。在Step1目录中创建一个CMakeLists.txt文件，如下所示:
  
     ```cmake
     cmake_minimum_required(VERSION 3.10)
     #set the project name
     project(Tutorial)
     #add the executable
     add_executable(Tutorial tutorial.cxx)
     ```
     ​	注意，这个例子在CMakeLists.txt文件中使用了小写命令。CMake支持大小写命令、大小写命令和混合大小写		命令。教程的源代码。cxx在Step1目录中提供，可用于计算一个数字的平方根。
  
     - 构建和运行
  
       这就是我们所需要的——我们现在就可以构建和运行我们的项目了!首先，运行cmake可执行文件或cmake-gui来配置项目，然后用您选择的构建工具构建它。例如，从命令行，我们可以导航到CMake源代码树的Help/guide/tutorial目录，并创建一个构建目录:
  
       ```shell
       mkdir Step1_build
       ```
     
       接下来，导航到构建目录并运行CMake来配置项目并生成原生构建系统:
     
       ```shell
       cd Step1_build
       cmake ../Step1
       ```
     
       **然后调用构建系统来实际编译/链接项目**:
     
       ```shell
       cmake --build .
       ```
     
     - 添加版本号和配置的头文件
     
       我们要添加的第一个特性是为我们的可执行文件和项目提供版本号。虽然我们可以在源代码中专门完成这一点，但使用CMakeLists.txt提供了更大的灵活性。
     
       首先，修改CMakeLists.txt文件，以使用project()命令设置项目名称和版本号。
     
       ```cmake
       cmake_minimum_required(VERSION 3.10)
       #set the project name and version
       project(Tutorial VERSION 1.0)
       ```
     
       然后，配置一个头文件将版本号传递给源代码:
     
       ```cmake
       configure_file(TutorialConfig.h.in TutorialConfig.h)
       ```
     
       由于配置的文件将被写入二叉树，因此必须将该目录添加到搜索包含文件的路径列表中。在CMakeLists.txt文件的末尾添加以下行:
     
       ```cmake
       target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
       ```
     
       使用您最喜欢的编辑器，在源目录中创建TutorialConfig.h.in，包含以下内容:
     
       ```c++
       // the configured options and settings for Tutorial
       #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
       #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
       ```
     
       当CMake配置这个头文件时，@Tutorial_VERSION_MAJOR@和@Tutorial_VERSION_MINOR@的值将被替换。
     
       下一个修改教程。cxx包含配置的头文件TutorialConfig.h。
     
       最后，让我们通过更新教程打印出可执行文件的名称和版本号。cxx如下:
     
       ```c++
         if (argc < 2) {
           // report version
           std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
                     << Tutorial_VERSION_MINOR << std::endl;
           std::cout << "Usage: " << argv[0] << " number" << std::endl;
           return 1;
         }
       ```
     
     - 指定c++标准
     
       接下来，让我们在tutorial.cxx中将atof替换为std::stod，从而为我们的项目添加一些c++ 11特性。同时，删除#include <cstdlib>。
     
       ```c++
       const double inputValue = std::stod(argv[1]);
       ```
     
       我们需要在CMake代码中显式地声明它应该使用正确的标志。在CMake中启用对特定c++标准的支持最简单的方法是使用CMAKE_CXX_STANDARD变量。对于本教程，将CMakeLists.txt文件中的CMAKE_CXX_STANDARD变量设置为11，将CMAKE_CXX_STANDARD_REQUIRED设置为True。确保在add_executable调用上面添加CMAKE_CXX_STANDARD声明。
     
     - 重新构建
     
       让我们重新构建我们的项目。我们已经创建了一个构建目录并运行了CMake，所以我们可以跳到构建步骤:
     
       ```shell
       cd Step1_build
       cmake --build .
       ```

2. <span id = "2">Step 2：添加Library</span>

   现在我们将向项目中添加一个库。这个库将包含我们自己的用于计算数字平方根的实现。然后，可执行程序可以使用这个库，而不是编译器提供的标准平方根函数。在本教程中，我们将把库放入名为MathFunctions的子目录中。这个目录已经包含了一个头文件MathFunctions.h和一个源文件mysqrt.cxx。源文件有一个名为mysqrt的函数，它提供了与编译器的sqrt函数类似的功能。将下面一行的CMakeLists.txt文件添加到MathFunctions目录中:

   ```cmake
   add_library(MathFunctions mysqrt.cpp)
   ```

   为了使用新的库，我们将在顶级的CMakeLists.txt文件中添加一个add_subdirectory()调用，以便构建库。我们将新库添加到可执行文件中，并将MathFunctions添加为包含目录，以便可以找到MathFunctions.h头文件。顶级CMakeLists.txt文件的最后几行现在应该是这样的:

   ```cmake
   #add the MathFunctions library
   add_subdirectory(MathFunctions)
   #add the executable
   add_executable(Tutorial tutorial.cxx)
   target_link_libraries(Tutorial PUBLIC MathFunctions)
   #add the binary tree to the search path for include files
   #so that we will find TutorialConfig.h
   target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}" "${PROJECT_BINARY_DIR}/MathFunctions")
   ```

   现在让我们将MathFunctions库设置为可选的。虽然在本教程中确实没有必要这样做，但对于较大的项目来说，这种情况很常见。第一步是向顶级CMakeLists.txt文件添加一个选项。

   ```cmake
   option(USE_MYMATH "Use tutorial provided math implementation" ON)
   #configure a header file to pass some of the CMake setting
   #to the source code
   configure_file(TutorialConfig.h.in TutorialConfig.h)
   ```

   该选项将显示在cmake-gui和ccmake中，默认值为ON，用户可以更改该值。该设置将存储在缓存中，这样用户就不需要每次在构建目录上运行CMake时都设置该值。下一个更改是使MathFunctions库的构建和链接成为条件。为此，我们将创建一个if语句来检查选项的值。在if块中，放入上面的add_subdirectory()命令和一些附加的列表命令，以存储链接到库所需的信息，并将子目录作为包含目录添加到教程目标中。顶级CMakeLists.txt文件的结尾现在看起来如下所示:

   ```cmake
   if(USE_MYMATH)
   	add_subdirectory(MathFunctions)
   	list(APPEND EXTRA_LIBS MathFunctions)
   	list(APPEND EXTRA_INCLUDES "${PEOJECT_SOURCE_DIR}/MathFunctions")
   endif()
   #add the executable
   add_executable(Tutorial tutorial.cxx)
   target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
   # so that we will find TutorialConfig.h
   target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}" "${EXTRA_INCLUDES}")
   ```

   

3. <span id = "3">Step 3：增加库的使用要求</span>

4. <span id = "4">Step 4：安装和测试</span>

5. <span id = "5">Step 5：添加系统自检</span>

6. <span id = "6">Step 6：添加自定义命令和生成文件</span>

7. <span id = "7">Step 7：包装一个安装程序</span>

8. <span id = "8">Step 8：添加对测试仪表板的支持</span>

9. <span id = "9">Step 9：选择静态库或共享库</span>

10. <span id = "10">Step 10：添加生成器表达式</span>

11. <span id = "11">Step 11：增加导出配置</span>

12. <span id = "12">Step 12：打包、调试和发布</span>