
# CMakeLists详解

## CMakeLists变量篇

+ 语法 SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
+ 指令功能 : 用来显式的定义变量 例子 : SET (SRC_LST main.c other.c)
+ 说明: 用变量代替值，例子中定义 SRC_LST 代替后面的字符串
+ 使用：使用 ${VAR} 来获取变量的值
+ 使用 ${NAME} 来获取变量的名称

## 参考地址
https://aiden-dong.github.io/2019/07/20/CMake%E6%95%99%E7%A8%8B%E4%B9%8BCMake%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%BA%94%E7%94%A8/