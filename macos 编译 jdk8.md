## 背景
我们平时在做 java 项目的时候，不可避免的要 debug 项目，在 debug 的时候，当碰到 java 自带的一些类的时候，我们需要查看一些变量，但是这个时候会碰到 ```local variable value is not available``` ，出现这个错误的原因是 java 自带的类在编译的时候没有加上 ```-g``` ，也就是没有加上调试信息，解决这个问题的一个方法就是重新编译成 debug 版本的 jdk。

## 编译

#### download

```
$ hg clone http://hg.openjdk.java.net/jdk8u/jdk8u jdk8u
$ cd jdk8u 
$ bash ./get_source.sh
```

#### configure 

```
# 需要提前安装 freetype ，自己编译或者使用 port 等都可以，我这里使用的是 port
bash configure --enable-debug --with-freetype-lib=/opt/local/lib/ --with-freetype-include=/opt/local/include/freetype2/
```

#### make

```
make CONF=macosx-x86_64-normal-server-fastdebug
make CONF=macosx-x86_64-normal-server-fastdebug images
```

## 遇到的问题

### The xxx compiler (located as xxx ) does not seem to be the required GCC compiler

解决方案：

```

# 使用 clang 编译，因为底层有些代码是基于 objective-c 的，默认自带的 gcc 是不支持的
$ export USE_CLANG=true
$ export CC=clang
$ export CXX=clang++


# 修改代码，将报错的地方注释掉
# 报错的原因仅仅是 clang --version 输出的结果没有 Free Software Foundation 这个字符串，可以忽略

diff --git a/jdk8u/common/autoconf/generated-configure.sh b/jdk8u/common/autoconf/generated-configure.sh
index 0025b6b..d76fc6f 100644
--- a/jdk8u/common/autoconf/generated-configure.sh
+++ b/jdk8u/common/autoconf/generated-configure.sh
@@ -20224,13 +20224,13 @@ $as_echo "$as_me: The result from running with -V was: \"$COMPILER_VERSION_TEST\
     COMPILER_VERSION_TEST=`$COMPILER --version 2>&1 | $HEAD -n 1`
     # Check that this is likely to be GCC.
     $COMPILER --version 2>&1 | $GREP "Free Software Foundation" > /dev/null
-    if test $? -ne 0; then
-      { $as_echo "$as_me:${as_lineno-$LINENO}: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&5
-$as_echo "$as_me: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&6;}
-      { $as_echo "$as_me:${as_lineno-$LINENO}: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&5
-$as_echo "$as_me: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&6;}
-      as_fn_error $? "GCC compiler is required. Try setting --with-tools-dir." "$LINENO" 5
-    fi
+    #if test $? -ne 0; then
+    #  { $as_echo "$as_me:${as_lineno-$LINENO}: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&5
+#$as_echo "$as_me: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&6;}
+    #  { $as_echo "$as_me:${as_lineno-$LINENO}: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&5
+#$as_echo "$as_me: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&6;}
+#$as_echo "$as_me: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&6;}
+    #  as_fn_error $? "GCC compiler is required. Try setting --with-tools-dir." "$LINENO" 5
+    #fi

     # First line typically looks something like:
     # gcc (Ubuntu/Linaro 4.5.2-8ubuntu4) 4.5.2
@@ -21825,13 +21825,13 @@ $as_echo "$as_me: The result from running with -V was: \"$COMPILER_VERSION_TEST\
     COMPILER_VERSION_TEST=`$COMPILER --version 2>&1 | $HEAD -n 1`
     # Check that this is likely to be GCC.
     $COMPILER --version 2>&1 | $GREP "Free Software Foundation" > /dev/null
-    if test $? -ne 0; then
-      { $as_echo "$as_me:${as_lineno-$LINENO}: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&5
-$as_echo "$as_me: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&6;}
-      { $as_echo "$as_me:${as_lineno-$LINENO}: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&5
-$as_echo "$as_me: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&6;}
-      as_fn_error $? "GCC compiler is required. Try setting --with-tools-dir." "$LINENO" 5
-    fi
+    #if test $? -ne 0; then
+    #  { $as_echo "$as_me:${as_lineno-$LINENO}: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&5
+#$as_echo "$as_me: The $COMPILER_NAME compiler (located as $COMPILER) does not seem to be the required GCC compiler." >&6;}
+    #  { $as_echo "$as_me:${as_lineno-$LINENO}: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&5
+#$as_echo "$as_me: The result from running with --version was: \"$COMPILER_VERSION_TEST\"" >&6;}
+    #  as_fn_error $? "GCC compiler is required. Try setting --with-tools-dir." "$LINENO" 5
+    #fi

```

### Making adlc Undefined symbols for architecture x86_64

具体错误：

```
Making adlc
Undefined symbols for architecture x86_64:
  "std::ostream::operator<<(int)", referenced from:
      printline(std::ostream&, char const*, int, char const*, int, int) in filebuff.o
  "std::ios_base::Init::Init()", referenced from:
      ___cxx_global_var_init in adlparse.o
      ___cxx_global_var_init in archDesc.o
      ___cxx_global_var_init in arena.o
      ___cxx_global_var_init in dfa.o
      ___cxx_global_var_init in dict2.o
      ___cxx_global_var_init in filebuff.o
      ___cxx_global_var_init in forms.o
      ...
  "std::ios_base::Init::~Init()", referenced from:
      ___cxx_global_var_init in adlparse.o
      ___cxx_global_var_init in archDesc.o
      ___cxx_global_var_init in arena.o
      ___cxx_global_var_init in dfa.o
      ___cxx_global_var_init in dict2.o
      ___cxx_global_var_init in filebuff.o
      ___cxx_global_var_init in forms.o
      ...
  "std::basic_ostream<char, std::char_traits<char> >& std::operator<<<std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)", referenced from:
      FileBuffRegion::print(std::ostream&) in filebuff.o
      printline(std::ostream&, char const*, int, char const*, int, int) in filebuff.o
  "std::basic_ostream<char, std::char_traits<char> >& std::operator<<<std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char)", referenced from:
      printline(std::ostream&, char const*, int, char const*, int, int) in filebuff.o
      expandtab(std::ostream&, int, char, char, char) in filebuff.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[8]: *** [../generated/adfiles/adlc] Error 1
```

解决方案：添加 编译参数 LFLAGS += -stdlib=libstdc++ ,具体的原因 [undefined symbol when build hotspot with Xcode5](https://bugs.openjdk.java.net/browse/JDK-8033898?page=com.atlassian.jira.plugin.system.issuetabpanels:all-tabpanel)

```
diff --git a/jdk8u/hotspot/make/bsd/makefiles/gcc.make b/jdk8u/hotspot/make/bsd/makefiles/gcc.make
index 840148e..91cbe22 100644
--- a/jdk8u/hotspot/make/bsd/makefiles/gcc.make
+++ b/jdk8u/hotspot/make/bsd/makefiles/gcc.make
@@ -365,6 +365,10 @@ ASFLAGS += -x assembler-with-cpp
 # statically link libstdc++.so, work with gcc but ignored by g++
 STATIC_STDCXX = -Wl,-Bstatic -lstdc++ -Wl,-Bdynamic

+ifeq ($(USE_CLANG), true)
+  LFLAGS += -stdlib=libstdc++
+endif
+
 ifeq ($(USE_CLANG),)
   # statically link libgcc and/or libgcc_s, libgcc does not exist before gcc-3.x.
   ifneq ("${CC_VER_MAJOR}", "2"
```

### error: '&&' within '||' [-Werror,-Wlogical-op-parentheses]

解决方案：

```
# 原因是检查太严格了
$ export COMPILER_WARNINGS_FATAL=false
```