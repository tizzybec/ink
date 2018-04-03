title: 使用premake5进行项目生成
date: 2016-05-06 10:52:00 +0800
update: 2016-05-06 10:52:00 +0800
author: me
#cover: -/images/default.png
tags:
    - premake
    - premake5
    - Visual Studio
    - cmake
    
---

最近在把之前写的YASS(Yet Another Simlation System)项目拆成三个部分，分别是tbcore、tbui和tbengine，YASS在win下使用vs2015（早期是2010）开发，构建系统是msbuild，这次拆分索性把项目生成这块好好做一下，开始考虑cmake，但看过文档后对cmake的dsl实在是望而生畏，所以只有使用经验没有编写经验，premake可以作为一个非常优秀的选择（不少开源项目在用）。

<!--more-->

premake已经到5.0 Alpha版本，离正式发布应该不远了，暂时不支持Xcode、MonoDevelop、 Code::Blocks和CodeLite，等到是5.0正式发布的时候这些特性都会补全（5.0之前都是支持的）。

使用premake基于如下几条原因:

1. 对cmake的dsl实在无感，不如来门正儿八经的脚本痛快
2. premake使用lua编写配置，足够简单，容易扩展
3. 单执行文件，轻量，没有cmake笨重
4. 支持的格式够用，vs2008~2005，GNU Make（包括Cygwin和MinGW），语言方面支持C、C++和C#
5. 完善参考文档，初次上手使用起来就觉得非常简单，而且功能强大

如果需要使用qt，[premake-qt](https://github.com/tizzybec/premake-qt)这个扩展能够实现，我在公司写了120+的项目生成脚本（为了兼容已有规则，400行左右），已经验证过其可用性（需要修改包导入的方式，可以看错误修改）。

几个与vs相关的参数：

1. 运行时库(/MD,/MDd),[runtime](https://github.com/premake/premake-core/wiki/runtime)
2. 项目字符集(Unicode, MBCS),[characterset](https://github.com/premake/premake-core/wiki/characterset)

几个坑:

1. excludes会覆盖
2. configuration废弃，建议使用filter，filter使用完以filter {} 结尾，关闭filter作用域
3. 注意global、workspace和project之间的作用域覆盖情况
4. 使用预编译头文件的时候需要写相对或者绝对路径，因为生成vs项目是使用的abspath进行比对，否则会出现预编译头文件无法创建的问题

最后附带为我的个人项目[tbcore](http://git.oschina.net/tizzybec/tbcore)编写的[premake文件](http://git.oschina.net/tizzybec/tbcore/blob/master/premake5.lua?dir=0&filepath=premake5.lua&oid=429836ab0f3c2dcabeaf41708e78b46af1999542&sha=4eaed5dc8cd782ecca595bcdefcf318349aecd75):

```lua
--
-- tbcore 1.0 build script 
--

------------------------------------------------------------

workspace "All"
  platforms { "x86", "x86_64" }
  configurations { "debug", "release" }
  location "build"
  
------------------------------------------------------------

project "tbcore"
  kind "SharedLib"
  language "C++"
  targetdir "build"
  objdir "%{cfg.location}/%{cfg.platform}/%{cfg.buildcfg}"
  
  defines 
  {
    "TB_BASE_EXPORTS", 
    "TB_GEO_EXPORTS", 
    "TB_MATH_EXPORTS", 
    "TB_ARCHIVE_EXPORTS", 
    "TB_NETWORK_EXPORTS", 
    "TB_DB_EXPORTS", 
    "TB_UTILS_EXPORTS", 
    "TB_REFLECTION_EXPORTS", 
    "TB_RUNTIME_EXPORTS"
   }
  
  filter { "platforms:x86" }
    architecture "x86"
  filter { }
  
  filter { "platforms:x86_64" }
    architecture "x86_64"
  filter { }
  
  filter { "configurations:Debug" }
    defines { "DEBUG", "_DEBUG"}
    targetname "tbcore-1.0-d"
    runtime "Debug"
  filrer { }
    
  filter "configurations:release"
    defines { "NDEBUG" }
    optimize "Speed"
    runtime "Release"
    targetname "tbcore-1.0"
  filter {}
  
  files 
  { 
    "./src/**.hpp", 
    "./src/**.cpp", 
    "./src/**.inl",
    "./include/**.hpp",
  }
    
  libdirs 
  { 
    "./3rdparty/lib/%{cfg.platform}/%{cfg.buildcfg}"
  }
    
  excludes 
  { 
    "**_test.cpp" 
   }
    
  includedirs 
  {
    "./3rdparty/include"
  }
   
  links 
  { 
    "srm.lib", 
    "rocksdb.lib" 
  }

------------------------------------------------------------

project "tbcore_test"
  kind "ConsoleApp"
  
  files 
  { 
    "**_test.cpp", 
    "src/testing/gmock/src/gmock_main.cc"
  }
```

### 参考资料

1. premake5 wiki[https://github.com/premake/premake-core/wiki]
2. lua 5.1 manual[http://www.lua.org/manual/5.1/manual.html]