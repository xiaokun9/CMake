# cmake(1)

1. <span id="1">概要</span>

     [Generate a Project Buildsystem](#4)
        	cmake [<options>] <path-to-source>
      	 cmake [<options>] <path-to-existing-build>
        	cmake [<options>] -S <path-to-source> -B <path-to-build>
   
   [Build a Project](#5)
   	 cmake --build <dir> [<options>] [-- <build-tool-options>]
   
   [Install a Project](#6)
    	cmake --install <dir> [<options>]
   
   [Open a Project](#7)
    	cmake --open <dir>
   
   [Run a Script](#8)
    	cmake [{-D <var>=<value>}...] -P <cmake-script-file>
   
   [Run a Command-Line Tool](#9)
    	cmake -E <command> [<options>]
   
   [Run the Find-Package Tool](#10)
    	cmake --find-package [<options>]
   
   [View Help](#11)
    	cmake --help[-<topic>]
   
2. 描述

   cmake可执行文件是跨平台构建系统生成器cmake的命令行界面。上面的[概要](#1)列出了该工具可以执行的各种操作，如下面的章节所述,

   要用CMake构建一个软件项目，[生成一个项目构建系统](#4),可以选择使用cmake来[构建项目](#5)，[安装项目](#6)，或者直接运行相应的构建工具(例如make)。cmake还可以用于[查看帮助](#11)

   其他操作用于软件开发人员使用[CMake语言](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#manual:cmake-language(7))编写脚本以支持他们的构建

   关于可以用来代替cmake的图形用户界面，请参阅[ccmake](https://cmake.org/cmake/help/latest/manual/ccmake.1.html#manual:ccmake(1))和[cmake-gui](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1))。有关CMake测试和打包工具的命令行接口，请参阅[ctest](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1))和[cpack](https://cmake.org/cmake/help/latest/manual/cpack.1.html#manual:cpack(1))。

   有关CMake的更多信息，请参见本手册末尾的[参考](#13)

3. <span id= "3">CMake构建系统介绍</span>

   构建系统描述了如何使用构建工具从源代码中构建项目的可执行程序和库，从而实现整个过程的自动化。例如，构建系统可能是与命令行make工具一起使用的Makefile，也可能是集成开发环境(IDE)的项目文件。为了避免维护多个这样的构建系统，项目可以使用[CMake语言](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#manual:cmake-language(7))编写的文件抽象地指定其构建系统。从这些文件中，CMake通过一个称为生成器的后端为每个用户在本地生成首选构建系统。

   要用CMake生成一个构建系统，必须选择以下选项:

   - 源代码树

     包含项目提供的源文件的顶级目录。该项目使用cmake-language(7)手册中描述的文件指定其构建系统，从名为CMakeLists.txt的顶级文件开始。这个文件指定构建目标和它们的依赖关系，如cmake-buildsystem(7)手册所述。

   - 构建树

     用于存储构建系统文件和构建输出工件(例如可执行文件和库)的顶级目录。CMake将编写一个CMakeCache.txt文件，将该目录标识为构建树，并存储诸如构建系统配置选项之类的持久信息。

     要维护原始源代码树，请使用单独的专用构建树执行源外构建。也支持将构建树放在与源树相同的目录中的内源代码构建，但不鼓励这样做

   - 生成器

     这将选择生成的构建系统的类型。有关所有生成器的文档，请参阅cmake-generators(7)手册。运行cmake——help查看本地可用的生成器列表。可以选择使用下面的-G选项来指定生成器，或者直接接受当前平台的默认CMake选择。

     当使用命令行构建工具生成器之一时，CMake期望编译器工具链所需的环境已经在shell中配置好了。当使用一个IDE构建工具生成器时，不需要特定的环境。

4. <span id = "4">生成一个项目构建系统</span>

   使用以下命令签名之一运行CMake以指定源代码并构建树并生成一个构建系统:

   ```
   cmake [<options>] <path-to-source>
   ```

   使用当前工作目录作为构建树，使用<path-to-source>作为源树。指定的路径可以是绝对的，也可以是相对于当前工作目录的。源树必须包含CMakeLists.txt文件，**不能包含CMakeCache.txt文件，因为后者标识了一个现有的构建树**。例如:

   ```shell
   mkdir build
   cd build
   cmake ../src
   ```

   ```
   cmake [<options>] <path-to-existing-build>
   ```

   使用<path-to-existing-build>作为构建树，并从其CMakeCache.txt文件中加载到源树的路径，该文件必须已经由CMake之前的运行生成。指定的路径可以是绝对的，也可以是相对于当前工作目录的。例如:

   ```shell
   cd build
   cmake .
   ```

   ```
   cmake [<options>] -S <path-to-source> -B <path-to-build>
   ```

   使用<path-to-build>作为构建树，使用<path-to-source>作为源树。指定的路径可以是绝对路径，也可以是相对于当前工作目录的路径。源树必须包含一个CMakeLists.txt文件。如果构建树还不存在，将自动创建它。例如:

   ```shell
   cmake -S src -B build
   ```

   在所有情况下，<选项>可能是下面选项的零个或多个。

   上面指定源代码和构建树的样式可以混合使用。用-S或-B指定的路径总是分别被分类为源树或构建树。使用纯参数指定的路径根据其内容和前面给出的路径类型进行分类。如果只给出了一种类型的路径，则使用当前工作目录(cwd)用于另一种路径。例如:

   | Command Line  | Source Dir    | Build Dir |
   | :--- | ---- | ---- |
   | cmake src | src | cwd |
   | cmake build(existing)|loaded |build |
   | cmake -S src|src| cwd|
   | cmake -S src build|src |build |
   | cmake -S src -B build|src |build |
   | cmake -B build|cwd |build |
   | cmake -B build src|src |build |
   | cmake -B build -S|src |build |
   
   在3.23版更改:CMake在指定多个源路径时发出警告。这从未被正式记录或支持过，但是较老的版本意外地接受了多个源路径并使用指定的最后一个路径。避免传递多个源路径参数。
   
   生成构建系统后，可以使用相应的本地构建工具来构建项目。例如，在使用Unix Makefiles生成器后，可以直接运行make:
   
   ```shell
   make
   make install
   ```
   
   或者，您也可以使用cmake通过自动选择和调用适当的本机构建工具来构建项目。
   
   > **Options:**
   
   ```
   -S <path-to-source>
   ```
   
   要构建的CMake项目根目录的路径。
   
   ```
   -B <path-to-build>
   ```
   
   CMake将用作构建目录根的目录的路径。如果目录不存在，CMake将创建它。
   
   ```
   -C <initial-cache>
   ```
   
   预加载一个脚本来填充缓存。
   
   当CMake第一次在一个空的构建树中运行时，它会创建一个CMakeCache.txt文件，并为项目填充可定制的设置。此选项可用于指定在第一次通过项目的CMake列表文件之前从其中加载缓存项的文件。加载的条目优先于项目的默认值。给定的文件应该是包含使用CACHE选项的set()命令的CMake脚本，而不是缓存格式文件。
   
   脚本中对**CMAKE_SOURCE_DIR**和**CMAKE_BINARY_DIR**的引用计算顶级源代码和构建树
   
   ```
   -D <var>:<type>=<value>,-D <var>=<value>
   ```
   
   创建或更新一个CMake CACHE条目。
   
   当CMake第一次在一个空的构建树中运行时，它会创建一个CMakeCache.txt文件，并为项目填充可定制的设置。此选项可用于指定优先于项目默认值的设置。该选项可以根据需要对任意数量的CACHE条目重复。
   
   如果给出了:<type>部分，它必须是set()命令文档为其CACHE签名指定的类型之一。如果省略:<type>部分，则创建的条目将不带类型(如果它不存在类型)。如果项目中的某个命令将类型设置为PATH或FILEPATH，则<值>将被转换为绝对路径。
   
   该选项也可以作为单个参数给出:``` -D <var>:<type>=<value>``` 或者 ``` -D <var>=<value>```
   
   ```
   -U <globbing_expr>
   ```
   
   从CMake缓存中删除匹配项。
   
   此选项可用于从CMakeCache.txt文件中删除一个或多个变量，使用*和?都受支持。该选项可以根据需要对任意数量的CACHE条目重复。
   
   **小心使用，你可以使你的CMakeCache.txt不能工作。**
   
   ```
   -G <generator-name>
   ```
   
   指定一个构建系统生成器
   
   CMake可能支持某些平台上的多个本机构建系统。生成器负责生成特定的构建系统。可能的生成器名称在cmake-generators(7)手册中指定。
   
   如果没有指定，CMake检查CMAKE_GENERATOR环境变量，否则退回到内置的默认选择
   
   ```
   -T <toolset-spec>
   ```
   
   生成器的工具集规范，如果支持的话。
   
   一些CMake生成器支持工具集规范，以告诉本机构建系统如何选择编译器。详细信息请参见CMAKE_GENERATOR_TOOLSET变量。
   
   ```
   -A <platform-name>
   ```
   
   如果生成器支持，请指定平台名称。
   
   一些CMake生成器支持平台名称，提供给本机构建系统以选择编译器或SDK。详细信息请参见CMAKE_GENERATOR_PLATFORM变量。
   
   ```
   --toolchain <path-to-file>
   ```
   
   指定交叉编译工具链文件，相当于设置CMAKE_TOOLCHAIN_FILE变量
   
   ```
   --install-prefix <directory>
   ```
   
   指定CMAKE_INSTALL_PREFIX变量使用的安装目录。必须是绝对路径。
   
   ```
   -Wno-dev
   ```
   
   抑制开发者警告
   
   禁止针对CMakeLists.txt文件作者的警告。默认情况下，这也将关闭弃用警告。
   
   ```
   -Wdev
   ```
   
   打开开发者警告
   
   启用针对CMakeLists.txt文件作者的警告。默认情况下，这也会打开弃用警告。
   
   ```
   -Werror=dev
   ```
   
   打开开发人员警告错误。
   
   打开针对CMakeLists.txt文件作者的警告出错。默认情况下，这也将打开已弃用的警告作为错误。
   
   ```
   -Wno-error=dev
   ```
   
   关闭开发人员警告错误。
   
   将针对CMakeLists.txt文件作者的警告设置为错误。默认情况下，这也将关闭已弃用的警告作为错误。
   
   ```
   -Wdeprecated
   ```
   
   启用已弃用的功能警告
   
   为使用已弃用的功能启用警告，这是针对CMakeLists.txt文件的作者的。
   
   ```
   -Wno-deprecated
   ```
   
   抑制过时的功能警告。
   
   禁止使用已弃用功能的警告，这是针对CMakeLists.txt文件的作者的。
   
   ```
   -Werror=deprecated
   ```
   
   使已弃用的宏和函数警告错误。
   
   对已弃用的宏和函数的使用发出警告，这意味着CMakeLists.txt文件的作者，错误
   
   ```
   -Wno-error=deprecated
   ```
   
   发出已弃用的宏和函数警告而不是错误。
   
   针对已弃用宏和函数的使用发出警告，这是针对CMakeLists.txt文件的作者的，而不是错误。
   
   ```
   --fresh
   ```
   
   3.24新版功能
   
   执行构建树的新配置。这将删除所有现有的CMakeCache.txt文件和相关的CMakeFiles/目录，并从头重新创建它们。
   
   ```
   -L[A][H]
   ```
   
   列出非高级缓存变量
   
   列表缓存变量将运行CMake并列出CMake缓存中所有未标记为INTERNAL或ADVANCED的变量。这将有效地显示当前的CMake设置，然后可以通过-D选项进行更改。更改一些变量可能会导致创建更多的变量。如果指定了A，那么它也将显示高级变量。如果指定了H，它还将显示每个变量的帮助。
   
   ```
   -N
   ```
   
   视图模式。
   
   只加载缓存。不实际运行configure并生成步骤。
   
   ```
   --graphviz=[file]
   ```
   
   生成依赖项的graphviz，参见CMakeGraphVizOptions了解更多。
   
   生成一个graphviz输入文件，该文件将包含项目中的所有库和可执行依赖项。有关更多细节，请参阅CMakeGraphVizOptions的文档。
   
   ```
   --system-information [file]
   ```
   
   Dump系统信息
   
   Dump关于当前系统的广泛信息。如果从CMake项目的二叉树顶部运行，它将dump额外的信息，如缓存、日志文件等。
   
   ```
   --log-level=<ERROR|WARNING|NOTICE|STATUS|VERBOSE|DEBUG|TRACE>
   ```
   
   设置log等级
   
   message()命令将只输出指定的日志级别或更高级别的消息。缺省日志级别为STATUS。
   
   要在CMake运行之间保持日志级别，可以将CMAKE_MESSAGE_LOG_LEVEL设置为缓存变量。如果同时给出了命令行选项和变量，则命令行选项优先。
   
   出于向后兼容的原因，```——loglevel```也被认为是该选项的同义词
   
   ```
   --log-context
   ```
   
   启用message()命令输出附加到每个消息的上下文。
   
   该选项只显示当前CMake运行的上下文。要使显示上下文对所有后续的CMake运行是持久的，可以将CMAKE_MESSAGE_CONTEXT_SHOW设置为缓存变量。当给出这个命令行选项时，CMAKE_MESSAGE_CONTEXT_SHOW将被忽略。
   
   ```
   --debug-trycompile
   ```
   
   不删除try_compile()构建树。一次只对一个try_compile()有用
   
   不删除为try_compile()调用创建的文件和目录。这在调试失败的try_compilation时非常有用。然而，它可能会改变try-编译的结果，因为来自前一次try-编译的旧垃圾可能会导致不同的测试通过或错误地失败。此选项最好用于一次一个try-compile，并且仅在调试时使用。
   
   ```
   --debug-output
   ```
   
   将cmake设置为调试模式。
   
   在cmake运行期间打印额外的信息，比如带有message(SEND_ERROR)调用的堆栈跟踪。
   
   ```
   --debug-find
   ```
   
   将cmake find命令置于调试模式。
   
   在cmake运行期间打印额外的查找调用信息，达到标准错误。输出是为人类使用而设计的，而不是用于解析。请参见CMAKE_FIND_DEBUG_MODE变量，以调试项目中更本地的部分。
   
   ```
   --debug-find-pkg=<pkg>[,...]
   ```
   
   在调用find_package(<pkg>)时，将cmake find命令置于调试模式，其中<pkg>是给定的逗号分隔的区分大小写的包名列表中的一个条目。
   
   比如```——debug-find```，但是将作用域限制到指定的包。
   
   ```
   --debug-find-var=<var>[,...]
   ```
   
   当使用<var>作为结果变量调用时，将cmake find命令置于调试模式，其中<var>是给定逗号分隔列表中的一个条目。类似于——debug-find，但将作用域限制为指定的变量名。
   
   ```
   --trace
   ```
   
   将cmake设置为跟踪模式。
   
   打印所有的调用记录。
   
   ```
   --trace-expand
   ```
   
   将cmake设置为跟踪模式。
   
   像```——trace```，但变量是展开的。
   
   ```
   --trace-format=<format>
   ```
   
   将cmake置于跟踪模式并设置跟踪输出格式。
   
   <format>可以是以下值之一。
   
   ```human``` 以人类可读的格式打印每个跟踪行。这是默认格式。
   
   ```json-v1``` 将每一行打印为单独的JSON文档。每个文档由换行符(\n)分隔。可以保证JSON文档中不会出现换行符,JSON trace 格式为：
   
   ```json
   {
       "file": "/full/path/to/the/CMake/file.txt",
     	"line": 0,
     	"cmd": "add_executable",
     	"args": ["foo", "bar"],
     	"time": 1579512535.9687231,
     	"frame": 2,
     	"global_frame": 4
   }
   ```
   
   成员变量：
   
   ​	```file``` 调用函数的CMake源文件的完整路径。
   
   ​	```line``` 文件中函数调用开始的那行
   
   ​	```line_end``` 如果函数调用跨越多行，则该字段将设置为函数调用结束的那一行。如果函数调用跨越单行，则取消设						置此字段。该字段是在json-v1格式的小版本2中添加的。
   
   ​	```defer``` 当cmake_language(DEFER)延迟函数调用时出现的可选成员。如果存在，它的值是一个字符串，包含延迟				调用<id>。
   
   ​	```cmd``` 被调用函数的名称。
   
   ​	```args``` 所有函数参数的字符串列表。
   
   ​	```time``` 函数调用的时间戳(从epoch开始的秒数)。
   
   ​	```frame ```在当前正在处理的CMakeLists.txt上下文中，被调用函数的堆栈帧深度。
   
   ​	```global_frame``` 被调用函数的堆栈帧深度，在跟踪中涉及的所有CMakeLists.txt文件中全局跟踪。该字段是在json-							v1格式的小版本2中添加的。
   
   ​							此外，输出的第一个JSON文档包含当前主要版本和次要版本的版本键:
   
   ​							
   
   ```json
   {
     "version": {
       "major": 1,
       "minor": 2
     }
   }	
   ```
   
   ```version``` 表示JSON格式的版本。该版本具有遵循语义版本约定的主要和次要组件。
   
   ```
   --trace-source=<file>
   ```
   
    将cmake置于跟踪模式，但只输出指定文件的行
   
   允许多个选项
   
   ```
   --trace-redirect=<file>
   ```
   
   将cmake置于跟踪模式并将跟踪输出重定向到文件而不是stderr。
   
   ```
   --warn-uninitialized
   ```
   
   警告未初始化的值。
   
   当使用未初始化的变量时，打印一个警告。
   
   ```
   --warn-unused-vars
   ```
   
   什么也不做。在CMake 3.2及以下版本中，启用了关于未使用变量的警告。在CMake版本3.3到3.18中，这个选项被破坏了。在CMake 3.19及以上版本中，该选项已被删除。
   
   ```
   --no-warn-unused-cli
   ```
   
   不要警告命令行选项。
   
   不要查找在命令行上声明但没有使用的变量
   
   ```
   --check-system-vars
   ```
   
   查找系统文件中变量使用的问题。
   
   通常，未使用和未初始化的变量只在CMAKE_SOURCE_DIR和CMAKE_BINARY_DIR中搜索。这个标志告诉CMake对其他文件也发出警告
   
   ```
   --compile-no-warning-as-error
   ```
   
   忽略目标属性COMPILE_WARNING_AS_ERROR和变量CMAKE_COMPILE_WARNING_AS_ERROR，防止编译时将警告作为错误处理。
   
   ```
   --profiling-output=<path>
   ```
   
   与`——profiling-format`一起使用，以输出到给定的路径。
   
   ```
   --profiling-format=<file>
   ```
   
   启用CMake脚本以给定格式输出剖析数据。
   
   这有助于对执行的CMake脚本进行性能分析。应该使用第三方应用程序将输出处理为人类可读的格式。
   
   目前支持的值是:Google - Trace输出在谷歌跟踪格式，它可以解析的谷歌Chrome的about:tracing选项卡或使用一个插件的工具，如跟踪罗盘。
   
   ```
   --preset <preset>,--preset=<preset>
   ```
   
   从<path to source>/CMakePresets读取预设值。json和< path-to-source > / CMakeUserPresets.json。预设可以指定生成器和构建目录，以及要传递给CMake的变量和其他参数的列表。当前工作目录必须包含CMake预设文件。CMake GUI也可以识别CMakePresets。json和CMakeUserPresets。json文件。有关这些文件的详细信息，请参见cmake-presets(7)。
   
   在所有其他命令行选项之前读取预设。预设所指定的选项(变量、生成器等)都可以通过在命令行上手动指定来覆盖。例如，如果预设将名为MYVAR的变量设置为1，但用户使用-D参数将其设置为2，则首选值为2。
   
   ```
   --list-presets,--list-presets=<[configure|build|test|all]>
   ```
   
   列出可用的预设。如果没有指定选项，则只列出配置预置。当前工作目录必须包含CMake预设文件。
   
   
   
5. <span id= "5">构建项目</span>

     CMake提供一个命令行签名来构建一个已经生成的项目二叉树:

     ```
     cmake --build <dir>             [<options>] [-- <build-tool-options>]
     cmake --build --preset <preset> [<options>] [-- <build-tool-options>]
     ```

     它用以下选项抽象了一个原生构建工具的命令行界面:

     `--build <dir>` 要构建的项目二进制目录。这是必需的(除非指定了预设)，并且必须是第一个。

     `--preset <preset>,--preset=<preset>` 使用生成预设来指定生成选项。项目二进制目录是从configurePreset键推断			出来的。当前工作目录必须包含CMake预设文件。更多细节请参见预设。

     `--list-presets` 列出可用的生成预设。当前工作目录必须包含CMake预设文件。

     `--parallel [<jobs>], -j [<jobs>]` 构建时使用的最大并发进程数。如果省略<jobs>，则使用本机构建工具的默认		编号。如果设置了CMAKE_BUILD_PARALLEL_LEVEL环境变量，则在没有给出该选项时指定默认并行级别。

     ​		一些本地构建工具总是并行构建。<jobs>值为1可用于将作业限制为单个作业。

     `--target <tgt>..., -t <tgt>...` 生成<tgt>而不是默认目标。可以给出多个目标，用空格分隔。

     `--config <cfg>` 对于多配置工具，请选择“configuration <cfg>”。

     `--clean-first` 首先构建clean的目标，然后再构建。(如果只进行清理，请使用`——target clean`。)

     `--resolve-package-references=<on|off|only>` 在构建之前解析来自外部包管理器(例如NuGet)的远程包引用。当设		置为on(默认)时，将在构建目标之前恢复包。当设置为only时，将恢复包，但不会执行任何构建。当设置为off		时，将不会恢复任何包。如果目标没有定义任何包引用，则此选项不执行任何操作。这个设置可以在构建预设中		指定(使用resolvePackageReferences)。如果指定了此命令行选项，则将忽略预设设置。如果没有提供命令行参		数或预设选项，将计算一个特定于环境的缓存变量来决定是否应该执行包恢复。当使用Visual Studio生成器时，		使用VS_PACKAGE_REFERENCES属性定义包引用。使用NuGet恢复包引用。可以通过将CMAKE_VS_NUGET_PACKAGE_RESTORE变量设置为OFF来禁用它。

     `--user-stderr` 忽略了。行为在CMake >= 3.0中是默认的

     `--verbose,-v` 启用详细输出(如果支持)，包括要执行的构建命令。如果设置了VERBOSE环境变量或CMAKE_VERBOSE_MAKEFILE缓存变量，则可以省略此选项。

     `--` 将剩下的选项传递给本机工具。

     运行`cmake --build`，不带任何快速帮助选项。

     

6. <span id= "6">安装项目</span>

     CMake提供了一个命令行签名来安装已经生成的项目二叉树:

     ```
     cmake --install <dir> [<options>]
     ```

     在不使用生成的生成系统或本机生成工具而构建项目以运行安装之后，可以使用此方法。可选项是:

     `--install <dir>` 要安装的项目二进制目录。这是必须的，而且必须是首要的。

     `--config <cfg>` 对于多配置生成器，选择configuration <cfg>。

     `--component <comp>` 基于组件的安装。只安装组件<comp>。

     `--default-directory-permissions <permissions>` 默认目录安装权限。权限格式<u=rwx,g=rx,o=rx>。

     `prefix <prefix>` 覆盖安装前缀CMAKE_INSTALL_PREFIX。

     `--strip` 去除之前的安装

     `-v,--verbose` 启用详细输出,如果设置了VERBOSE环境变量，则可以省略此选项。

     运行`cmake --install`，不带任何快速帮助选项。

7. <span id= "7">打开项目</span>

     `cmake --open <dir>` 在关联的应用程序中打开生成的项目。这只被一些生成器支持。

8. <span id= "8">运行脚本</span>

     `cmake [{-D <var>=<value>}...] -P <cmake-script-file> [-- <unparsed-options>...]` 将给定的cmake文件作为用cmake语言编写的脚本处理。不执行配置或生成步骤，也不修改缓存。如果使用-D定义变量，则必须在-P参数之前完成此操作,在`——`之后的任何选项都不会被CMake解析，但它们仍然包含在传递给脚本的CMAKE_ARGV<n>变量集合中(包括`——`本身)。

9. <span id= "9">运行命令行工具</span>

     CMake通过签名提供内置的命令行工具

     `cmake -E <command> [<options>]` 执行`cmake -E`或`cmake -E help`查看命令的摘要。可用的命令是:

     `capabilities` 3.7新版功能。以JSON格式报告cmake功能。输出是一个带有以下键的JSON对象:

     ​		`version` 带有版本信息的JSON对象。键是:

     ​				`string` `cmake --version`显示的完整版本字符串。

     ​				`major` 以整数形式表示的主版本号。

     ​				`minor ` 整数形式的副版本号。

     ​				`patch ` 整数形式的补丁级别。

     ​				`suffix` cmake版本后缀字符串。

     ​				`isDirty` 如果cmake构建来自脏树，则设置该bool值

     ​		`generators` 一个可用生成器列表。每个生成器都是一个JSON对象，具有以下键:

     ​				`name` 包含生成器名称的字符串

     ​				`toolsetSupport` 如果生成器支持工具集，则为True，否则为false。

     ​				`platformSupport ` 如果生成器支持平台，则为True，否则为false。

     ​				`supportedPlatforms` 3.2.1新版功能。当生成器通过CMAKE_GENERATOR_PLATFORM (-A…)支持平台规															范时可能出现的可选成员。该值是已知受支持的平台的列表。

     ​				`extraGenerators` 包含与生成器兼容的所有额外生成器的字符串列表。

     ​		`fileApi` 当`cmake-file-api(7)`可用时出现的可选成员。该值是一个带有一个成员的JSON对象:

     ​				`requests` 包含0个或多个支持的文件api请求的JSON数组。每个请求都是一个带有成员的JSON对象

     ​						`kind ` 指定一个支持的对象类型。

     ​						`version` 一个JSON数组，它的每个元素都是一个JSON对象，包含主要成员和次要成员，指定非负整									数版本组件

     ​		`serverMode` 如果cmake支持服务器模式，则为True，否则为false。自CMake 3.20以来始终为false。

     `cat [--] <files>...` 3.18新版功能。连接文件并在标准输出上打印。3.24新版功能:增加了对双破折号参数`——`的支		持。cat的这个基本实现不支持任何选项，因此使用以-开头的选项将导致错误。如果文件以-开头，则使用`——`来		指示选项的结束。

     `chdir <dir> <cmd> [<arg>...]` 更改当前工作目录并运行命令。

     `compare_files [--ignore-eol] <file1> <file2>` 检查<file1>是否与<file2>相同。如果文件相同，则返回0，否		则返回1。在无效参数的情况下，它返回2。3.14新版功能:`——ignore-eol`选项暗示行级比较并忽略LF/CRLF差异

     `copy <file>... <destination>` 将文件复制到<目标>(文件或目录)。如果指定了多个文件，<destination>必须是		directory且必须存在。不支持通配符。复制遵循符号链接。这意味着它不复制符号链接，而是复制它所指向的文		件或目录。3.5新版功能:支持多个输入文件。

     `copy_directory <dir>... <destination>` 复制<dir>…目录到<目标>目录。如果<destination>目录不存在，它		将被创建。Copy_directory遵循符号链接。3.5新版功能:支持多个输入目录。3.15新版功能:当源目录不存在时，		命令会失败。以前它通过创建一个空的目标目录成功。

     `copy_if_different <file>... <destination>` 如果文件已经更改，则将文件复制到<destination>(文件或目录)。		如果指定了多个文件，<destination>必须是directory且必须存在。Copy_if_different跟随符号链接。

     `create_symlink <old> <new>` 创建一个符号链接<新>命名<旧>。3.13新版功能:支持在Windows上创建符号链接,**注		意**创建<new>符号链接的路径必须事先存在。

     `create_hardlink <old> <new>` 3.19新版功能。创建硬链接<新>命名<旧>。**注意**创建<new>硬链接的路径必须事先存		在。<old>必须事先存在。

     `echo [<string>...]` 将参数显示为文本

     `echo_append [<string>...]` 将参数显示为文本，但不显示新行。

     `env [--unset=name ...] [NAME=VALUE ...] [--] <command> [<arg>...]` 3.1新版功能。在修改后的环境中执行命		令。3.24新版功能:增加了对双破折号参数——的支持。使用——停止解释选项/环境变量，并将下一个参数视为		命令，即使它以-或包含=开头。

     `environment` 显示当前的环境变量

     `false` 3.16新版功能。什么都不做，退出代码为1。

     `make_directory <dir>...` 创建< dir >目录。如果有必要，也创建父目录。如果一个目录已经存在，它将被静默地忽		略。3.5新版功能:支持多个输入目录。

     `md5sum <file>...` 创建md5sum兼容格式的MD5校验和文件:

     `sha1sum <file>...` 创建SHA1校验和文件的sha1sum兼容格式:

     `sha224sum <file>...` 创建SHA224校验和文件的sha224sum兼容格式:

     `sha256sum <file>...` 创建SHA256校验和文件，格式为sha256sum兼容:

     `sha384sum <file>...` 创建SHA384校验和文件的sha384sum兼容格式:

     `sha512sum <file>...` 创建SHA512校验和文件，格式为sha512sum兼容:

     `remove [-f] <file>...` 3.17版后已移除。删除文件(s)。计划的行为是，如果列出的任何文件已经不存在，该命令返		回一个非零的退出代码，但不记录任何消息。-f选项改变行为，在这种情况下返回零退出码(即成功)。移除不跟随		符号链接。这意味着它只删除符号链接，而不删除它所指向的文件。实现有bug，总是返回0。如果不破坏向后兼		容性，就无法解决这个问题。使用rm。

     `remove_directory <dir>...` 3.17版后已移除。删除<dir>目录及其内容。如果一个目录不存在，它将被静默地忽		略。使用rm。3.15新版功能:支持多个目录。3.16新版功能:如果<dir>是指向某个目录的符号链接，则只删除符		号链接。

     `rename <oldname> <newname>` 重命名一个文件或目录(在一个卷上)。如果具有<newname>名称的文件已经存在，那么		它将被静默替换。

     `rm [-rRf] [--] <file|dir>...` 3.17新版功能。删除文件<file>或目录<dir>。使用-r或-R可递归删除目录及其内		容。如果列出的任何文件/目录不存在，该命令将返回一个非零的退出代码，但不记录任何消息。-f选项改变行		为，在这种情况下返回零退出码(即成功)。使用——停止解释选项并将所有剩余参数视为路径，即使它们以-开		头。

     `server`  发射cmake-server(7)模式。

     `sleep <number>...` 3.0新版功能。睡眠时间是固定的。

     `tar [cxt][vf][zjJ] file.tar [<options>] [--] [<pathname>...]` 创建或提取tar或zip归档文件。选项有:

     ​		`c` 创建一个包含指定文件的新存档。如果使用<pathname>…参数是强制性的。

     ​		`x` 从存档提取到磁盘。3.15新版功能:<pathname>…参数只能用于提取选定的文件或目录。当提取选定的文件				或目录时，必须提供它们的确切名称，包括由list (-t)打印的路径。

     ​		`t` 归档内容列表。3.15新版功能:<pathname>…参数可用于仅列出选定的文件或目录

     ​		`v` 生成详细输出。

     ​		`z` 使用gzip压缩生成的归档文件。

     ​		`j` 使用bzip2压缩生成的存档文件。

     ​		`J` 3.1新版功能。使用XZ压缩结果存档

     ​		`--zstd` 3.15新版功能。使用Zstandard压缩结果存档。

     ​		`--files-form=<file>` 3.1新版功能。从给定的文件中读取文件名，每行一个。空行被忽略。行不能以-开头，除				非——add-file=<name>添加名称以-开头的文件。

     ​		`--format=<format>` 3.3新版功能。指定要创建的存档的格式。支持的格式有:7zip、gnutar、pax、paxr(受限					pax，默认)、zip。

     ​		`--mtime=<date>`  3.1新版功能。指定在tarball条目中记录的修改时间。

     ​		`--touch` 3.24新版功能。使用当前本地时间戳，而不是从存档中提取文件时间戳。

     ​		`--` 3.1新版功能。停止解释选项，并将所有剩余参数视为文件名，即使它们以-开头。

     `time <command> [<args>...]` 执行命令并显示运行时间。3.5新版功能:该命令现在正确地将带有空格或特殊字符的参		数传递给子进程。这可能会破坏那些通过自己的额外引用或转义绕过bug的脚本。

     `touch <file>...` 如果文件不存在，则创建<文件>。如果<文件>存在，则它正在更改<文件>的访问和修改次数。

     `touch_nocreate <file>...` touch一个文件，如果它存在，但不创建它。如果一个文件不存在，它将被静默地忽略。

     `true` 3.16新版功能。什么都不做，退出码为0。

10. windows命令行工具

     以下cmake -E命令仅适用于Windows系统:

     `delete_regv <key>` 删除Windows注册表值。

     `env_vs8_wince <sdkname>` 3.2新版功能。显示一个批处理文件，为VS2005中安装的Windows CE SDK设置环境。

     `env_vs9_wince <sdkname>` 3.2新版功能。显示一个批处理文件，为VS2008中安装的Windows CE SDK设置环境。

     `write_regv <key> <value>` 写入Windows注册表值。

11. <span id= "10">运行查找包工具</span>

      CMake为基于makefile的项目提供了类似pkg-config的帮助器:

      `cmake --find-package [<options>]` 它使用find_package()搜索包并将结果标志打印到stdout。这可以代替pkg-config		在普通的基于makefile的项目或基于autoconf的项目中查找已安装的库(通过share/aclocal/cmake.m4)。由于一些		技术限制，该模式不太受支持。保留它是为了兼容性，但不应该在新项目中使用。

12. <span id= "11">查看帮助</span>

      要从CMake文档中打印选定的页面，请使用

      `cmake --help[-<topic>]` 使用以下选项之一:

      ​		`--help,-help,-usage,-h,-H,/?` 打印使用信息并退出。用法描述基本命令行接口及其选项。

      ​		`--version,-version,/V [<f>]` 显示程序名称/版本栏并退出。如果指定了一个文件，则会将版本写入其中。如果				给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-full [<f>]` 打印所有帮助手册并退出。所有手册都以人类可读的文本格式印刷。如果给出了帮助，则将				帮助打印到名为<f>ile的文件中。

      ​		`--help-manual <man> [<f>]` 打印一个帮助手册并退出。指定的手册以人类可读的文本格式打印。如果给出了帮				助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-manual-list [<f>]` 列出可用的帮助手册并退出。该列表包含所有可以通过使用——help-manual选项后				接手册名称获得帮助的手册。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-command <cmd> [<f>]` 打印一个命令的帮助并退出。<cmd>的cmake-commands(7)手动条目以人类可读				的文本格式打印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-command-list [<f>]` 列出具有可用帮助的命令并退出。该列表包含所有命令，可以通过使用后跟命令名				的——help-command选项获得帮助。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-commands [<f>]` 打印cmake命令手册并退出。cmake-commands(7)手册以人类可读的文本格式打印。				如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-module <mod> [<f>]` 打印一个模块的帮助并退出。<mod>的cmake-modules(7)手动条目以人类可读的文				本格式打印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-module-list [<f>]` 列出具有可用帮助的模块并退出。该列表包含所有可以通过使用——help-module选				项后跟模块名获得帮助的模块。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-modules [<f>]` 打印cmake-modules手册并退出。cmake-modules(7)手册以人类可读的文本格式打印。				如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-policy <cmp> [<f>]` 打印一个策略的帮助并退出。<cmp>的cmake-policies(7)手动条目以人类可读的文				本格式打印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-policy-list [<f>]` 列出具有可用帮助的策略并退出。该列表包含所有可以通过使用——help-policy选项				后接策略名称获得帮助的策略。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-policies [<f>]` 打印cmake-policies手册并退出。cmake-policies(7)手册以人类可读的文本格式打印。				如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-property <prop [<f>]>` 打印一个属性的帮助并退出。<prop>的cmake-properties(7)手动条目以人类可				读的文本格式打印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-property-list [<f>]` 列出带有可用帮助的属性并退出。该列表包含所有可以通过使用后面跟着属性名				的——help-property选项获得帮助的属性。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-properties [<f>]` 打印cmake-properties手册并退出。cmake-properties(7)手册以人类可读的文本格式				打印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-variable <var> [<f>]` 打印一个变量的帮助并退出。<var>的cmake-variables(7)手动条目以人类可读				的文本格式打印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-variable-list [<f>]` 列出具有可用帮助的变量并退出。该列表包含所有可以通过使用——help-variable				选项后跟变量名获得帮助的变量。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      ​		`--help-variables [<f>]` 打印cmake-variables手册并退出。cmake-variables(7)手册以人类可读的文本格式打				印。如果给出了帮助，则将帮助打印到名为<f>ile的文件中。

      若要查看项目可用的预置，请使用

      `cmake <source-dir> --list-presets`

13. <span id= "12">返回值</span>

      在常规终止时，cmake可执行程序返回退出代码0。

      如果终止是由命令消息(FATAL_ERROR)或其他错误条件引起的，则返回一个非零退出码。

14. <span id="13">参考</span>

      以下资源可用于使用CMake获得帮助:

      - [Home Page]([https://cmake.org](https://cmake.org/))
      - [Online Documentation and Community Resources](https://cmake.org/documentation)
      - [Discourse Forum]([https://discourse.cmake.org](https://discourse.cmake.org/))