CMake快速上手笔记
=================

> 前段时间在学习Qgis示例代码时，需要利用CMakeList.txt文件生成工程，生成工程中出现很多错误，想着打开CMakeList.txt看看，为了看懂就顺便学习了CMake的知识。


# 基础
CMake工程定义:  
````
PROJECT(projectname [C] [CXX] [Java])
````

两个隐藏的cmake变量：  
````
<projectname>_BINARY_DIR
<projectname>_SOURCE_DIR
````

另外有两个预定义变量:  
````
PROJECT_BINARY_DIR`
PROJECT_SOURCE_DIR`
````

这两个变量分别与前面两个变量对应，建议使用这个两个变量，因为当工程名修改后无需修改这里的变量名。

SET指令：显示定义变量  
语法：  
````
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
````

例如：  
````
SET(SRC_LIST main.c t1.c t2.c)
SET(SRC_LIST “main.c”)
````

MESSAGE指令：向终端输出用户定义的信息  
语法：  
````
MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display" ...)
````

* SEND_ERROR：产生错误，生成过程被跳过；
* STATUS：产生前缀为“-”的信息；
* FATAL_ERROR：立即终止所有cmake过程。
* ADD_EXECUTABLE指令：定义工程生成可执行文件的文件名 
 
例如：  
````
ADD_EXECUTABLE(hello ${SRC_LIST})
````

其中“${}”表示引用变量的值，在IF控制语句中不能加它，直接引用变量名。


# 外部构建(out-of-source)  
* ADD_SUBDIRECTORY：用于向当前工程添加存放源文件的子目录，并可以指定“中间二进制”和“目标二进制”存放位置
* ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])  
通过SET指令可以重新指定目标二进制的位置，例如：  
````
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
````


# 安装  
INSTALL指令、CMAKE_INSTALL_PREFIX变量：用于定义安装规则，安装的内容可以包括目标二进制、动态库、静态库以及文件、目录、脚本等。

## INSTALL目标文件安装  
语法：  
````
INSTALL(TARGETS targets...
		[[ARCHIVE|LIBRARY|RUNTIME]
			[DESTINATION <dir>]
			[PERMISSIONS permissions...]
			[CONFIGURATIONS
				[Debug|Release|...]]
			[COMPONENT <component>]
			[OPTIONAL]
		] [...])
````

TARGETS后面跟的就是通过 ADD_EXECUTABLE 或者 ADD_LIBRARY 定义的目标文件，可能是可执行二进制、动态库、静态库。
目标类型也就相对应的有三种，ARCHIVE 特指静态库，LIBRARY 特指动态库，RUNTIME特指可执行目标二进制。
DESTINATION 定义了安装的路径，如果路径以/开头，那么指的是绝对路径，这时候CMAKE_INSTALL_PREFIX 其实就无效了。如果你希望使用 CMAKE_INSTALL_PREFIX 来定义安装路径，就要写成相对路径，即不要以/开头，那么安装后的路径就是${CMAKE_INSTALL_PREFIX}/<DESTINATION 定义的路径>  
例如：  
````
INSTALL(TARGETS myrun mylib mystaticlib
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION libstatic
		)
````

上面的例子会将：  
可执行二进制 myrun 安装到${CMAKE_INSTALL_PREFIX}/bin 目录
动态库 libmylib 安装到${CMAKE_INSTALL_PREFIX}/lib 目录
静态库 libmystaticlib 安装到${CMAKE_INSTALL_PREFIX}/libstatic 目录

## INSTALL普通文件的安装  
````
INSTALL(FILES files... DESTINATION <dir>
		[PERMISSIONS permissions...]
		[CONFIGURATIONS [Debug|Release|...]]
		[COMPONENT <component>]
		[RENAME <name>] [OPTIONAL]
		)
````

可用于安装一般文件，并可以指定访问权限，文件名是此指令所在路径下的相对路径。如果默认不定义权限 PERMISSIONS，安装后的权限为：OWNER_WRITE, OWNER_READ, GROUP_READ,和 WORLD_READ，即 644 权限。

## 非目标文件的可执行程序安装(如脚本之类)  
````
INSTALL(PROGRAMS files... DESTINATION <dir>
		[PERMISSIONS permissions...]
		[CONFIGURATIONS [Debug|Release|...]]
		[COMPONENT <component>]
		[RENAME <name>] [OPTIONAL])
````

跟上面的 FILES 指令使用方法一样，唯一的不同是安装后权限为:OWNER_EXECUTE, GROUP_EXECUTE, 和 WORLD_EXECUTE，即 755 权限。

## INSTALL目录安装  
````
INSTALL(DIRECTORY dirs... DESTINATION <dir>
		[FILE_PERMISSIONS permissions...]
		[DIRECTORY_PERMISSIONS permissions...]
		[USE_SOURCE_PERMISSIONS]
		[CONFIGURATIONS [Debug|Release|...]]
		[COMPONENT <component>]
		[[PATTERN <pattern> | REGEX <regex>]
		[EXCLUDE] [PERMISSIONS permissions...]] [...]
		)
````

这里主要介绍其中的 DIRECTORY、PATTERN 以及 PERMISSIONS 参数。  
DIRECTORY 后面连接的是所在 Source 目录的相对路径，但务必注意：abc 和 abc/有很大的区别。如果目录名不以/结尾，那么这个目录将被安装为目标路径下的 abc，如果目录名以/结尾，代表将这个目录中的内容安装到目标路径，但不包括这个目录本身。PATTERN 用于使用正则表达式进行过滤，PERMISSIONS 用于指定 PATTERN 过滤后的文件权限。

