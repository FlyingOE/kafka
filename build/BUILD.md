# How to build Kafka support for kdb+/q (x86/x64)

## Background

[Apache Kafka&reg;](https://kafka.apache.org/) is a distributed streaming platform that:
- Publishes and subscribes to streams of records, similar to a message queue or enterprise messaging system.
- Stores streams of records in a fault-tolerant durable way.
- Processes streams of records as they occur.

Reference: [Get Started with Kafka](https://docs.confluent.io/current/kafka/)

[`krk`](https://github.com/KxSystems/kafka) is a thin wrapper for kdb+ around [`librdkafka`](https://github.com/edenhill/librdkafka) C API for Kafka, which is part of the [_Fusion for kdb+_](https://code.kx.com/v2/interfaces/fusion/) interface collection.

Reference: [Using Kafka with kdb+](https://code.kx.com/q/interfaces/kafka/)

## Problem Statement

`krk` provides a default script for building the library using Visual Studio 2017 on Windows x64 platform.

However, both the underlying `librdkafka` and `krk` itself are not trivial to install/build on Windows, due to the lack of a standardized development environment. Furthermore, one may want to build `krk` for x86 platform, which is not supported by the default build script.

## Build Steps

The following instructions assumes Visual Studio 2019 Community as the base environment, though it should not be difficult to adapt the steps to other versions of Visual Studio.

### Download `librdkafka`

`librdkafka` can be obtained from [NuGet](https://www.nuget.org/).

1. In CMD:
   ```bat
cd /d {krk_src_dir}
:: NuGet needs a project in order to install dependencies
md dummy.proj
```
2. With Visual Studio, create an empty C++ project in `dummy.proj` directory
3. Menu >> Tools >> NuGet Package Manager >> Package Manager Console
4. In the PM console:
   ```powershell
Install-Package librdkafka.redist -Version 1.5.2
```
   This should install `librdkafka.redist` into `packages\librdkafka.redist.1.5.2` directory.

Reference: [`librdkafka.redist`](https://www.nuget.org/packages/librdkafka.redist/)

### Prepare build directory

As `librdkafka`'s directory structure is rather complex, it'd be much easier if we copy the files we need for bulding `krk` into the `build` directory.

1. In CMD:
   ```bat
cd /d {krk_src_dir}
cd build
md librdkafka\x86 librdkafka\x64
```
2. Copy all header files in `dummy.proj\packages\librdkafka.redist.1.5.2\build\native\include\librdkafka` to `build\librdkafka` directory.
3. Copy all library files in `dummy.proj\packages\librdkafka.redist.1.5.2\build\native\lib\win\x86\win-x86-Release\v120` to `build\librdkafka\x86` directory.
4. Copy all library files in `dummy.proj\packages\librdkafka.redist.1.5.2\build\native\lib\win\x64\win-x64-Release\v120` to `build\librdkafka\x64` directory.

### Download kdb+ build dependencies

kdb+ build dependencies can be downloaded from [GitHub](https://github.com/KxSystems/kdb/).

1. In CMD:
   ```bat
cd /d {krk_src_dir}
cd build
md q\w32 q\w64
```
2. Download <https://github.com/KxSystems/kdb/raw/master/c/c/k.h> into `build\q` directory.
3. Download <https://github.com/KxSystems/kdb/raw/master/w32/q.lib> into `build\q\w32` directory.
4. Download <https://github.com/KxSystems/kdb/raw/master/w64/q.lib> into `build\q\w64` directory.

Reference: [`build.bat`](./build.bat)

### Build `krk`

#### 32-bit binary

1. Open Visual Studio developer command prompt:
   Start menu >> Visual Studio 2019 >> x86 Native Tools Command Prompt for VS 2019
2. ```bat
cd /d {krk_src_dir}
cd build
cl /LD /DKXVER=3 /I. /Iq /Felibkfk.dll /O2 ..\kfk.c q\w32\q.lib /link /LIBPATH:librdkafka\x86
```
3. ```bat
cd /d {krk_src_dir}
md bin\w32
xcopy dummy.proj\packages\librdkafka.redist.1.5.2\runtimes\win-x86\native\*.dll bin\w32
xcopy build\libkfk.dll bin\w32
```
4. ```bat
cd bin\w32
q ..\..\kfk.q
```

Reference: [`build.bat`](./build.bat)

#### 64-bit binary

Similar to the steps [above](#32-bit binary), but change directories `w32` to `w64` and `x86` to `x64`.