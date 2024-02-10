# 关于C语言+JavaScript 检查应用内存是否更改

要检测一个特定程序是否被注入或内存修改，可以使用Node.js来运行JavaScript代码，并通过一些扩展模块来调用C 编写的函数。

----
博客网站：[https://me.tianbeigm.cn/archives/MemoryDetection](https://me.tianbeigm.cn/archives/MemoryDetection)
----
### 目录

- [总言](#总言)
  - [C语言部分](#c语言部分)
  - [Node.js扩展](#nodejs扩展)
  - [JavaScript部分](#javascript-部分)
- [C语言检测内存修改的示例代码](#c语言检测内存修改的示例代码)
- [使用Node.js调用C语言模块](#使用nodejs调用c语言模块)
- [JavaScript 部分](#javascript-部分-1)
  - [步骤1：安装`ffi-napi`](#步骤1安装ffi-napi)
  - [步骤2：编写Node.js代码](#步骤2编写nodejs代码)
- [C语言部分](#c语言部分-1)
  - [C语言代码](#c语言代码)
  - [编译为共享库](#编译为共享库)
    - [在Linux上编译](#在linux上编译)
    - [在Windows上编译](#在windows上编译)
  - [使用Node.js调用](#使用nodejs调用)
- [Node.js 通信部分](#nodejs-通信部分)
  - [Node.js HTTP服务器](#nodejs-http服务器)
  - [预留通讯接口](#预留通讯接口)


## 总言

1. **C语言部分**：编写C语言代码，用于实际检测特定程序内存是否被修改。这可能涉及到读取目标程序的内存页，计算校验和，与预期的校验和进行比较，或者检测内存中是否存在特定的注入代码模式。

2. **Node.js扩展**：通过Node.js的`node-gyp`或`N-API`等工具，将C语言编写的内存检测功能封装为Node.js模块。

3. **JavaScript部分**：使用Node.js调用这个C语言编写的模块，实现用户界面和逻辑控制，以便用户可以指定要检测的程序，启动内存检测过程，并获取检测结果。

下面提供一个简化的示例来说明这个过程：

### C语言检测内存修改的示例代码

```c
// memory_check.c
#include <stdio.h>
#include <stdbool.h>

// 假设的检测函数
bool check_memory_modification() {
    // 实现检测逻辑
    // 示例
    printf("Checking memory for modifications...\n");
    
    // 检测内存修改
    bool is_modified = true;
    
    return is_modified;
}
```

### 使用Node.js调用C语言模块

首先，你需要使用`node-gyp`将上述C代码编译为Node.js可加载的模块。然后，你可以编写如下的JavaScript代码来加载这个模块并调用其中的函数：

```javascript
// 使用Node.js的`ffi-napi`库来加载C语言编写的动态链接库（DLL或.so）
const ffi = require('ffi-napi');

// 假设你已经将上面的C代码编译为名为`memory_check_lib`的动态链接库
const lib = ffi.Library('./memory_check_lib', {
  'check_memory_modification': ['bool', []]
});

// 调用C语言函数
const isModified = lib.check_memory_modification();
console.log("Is memory modified?", isModified);
```

## JavaScript 部分

要完整地实现一个Node.js脚本，我们将模拟调用一个C语言函数的过程，这个函数检查内存是否被修改。为此，我们需要先假设C语言部分已经被编译成了一个共享库（例如，在Windows上是`.dll`文件，在Linux或macOS上是`.so`文件），且这个库提供了一个名为`check_memory_modification`的函数。

在实际应用中，你需要根据具体的C代码调整Node.js代码，确保方法签名和类型匹配。这里，我们使用`ffi-napi`库从Node.js调用C语言函数，因为`ffi-napi`允许Node.js代码调用任意的本地函数。

### 步骤1：安装`ffi-napi`

首先，你需要安装`ffi-napi`。在你的Node.js项目目录下，运行以下命令：

```sh
npm install ffi-napi
```

### 步骤2：编写Node.js代码

假设你的C库名为`libmemory_check.so`（或`memory_check.dll`），并且该库及其函数声明如下所示：

- 函数`check_memory_modification`没有参数，并返回一个布尔值表示内存是否被修改。

以下是调用该函数的Node.js代码：

```javascript
const ffi = require('ffi-napi');

// 定义C库的路径和函数原型
let libraryPath = './libmemory_check.so'; // 或者在Windows上使用 './memory_check.dll'
let memoryCheckLib = ffi.Library(libraryPath, {
  'check_memory_modification': ['bool', []],
});

function checkMemoryModification() {
  try {
    // 调用C语言函数
    let isModified = memoryCheckLib.check_memory_modification();
    console.log("Memory modification check result:", isModified ? "Modified" : "Not Modified");
  } catch (error) {
    console.error("Error calling C function:", error);
  }
}

// 执行检查
checkMemoryModification();
```

### 注意

- 请确保`libraryPath`变量正确指向了你的C库文件。
- 如果你的C函数需要参数，你需要在`ffi.Library`调用中适当调整参数类型。
- 这个示例假设`check_memory_modification`函数立即返回检查结果。

## C语言部分

为了创建一个C语言库，我们将编写一个示例函数`check_memory_modification`，该函数模拟检查程序内存是否被修改。然后，我们将编译这个C代码为一个共享库（`.dll`或`.so`文件），使得Node.js脚本可以通过`ffi-napi`调用它。

### C语言代码

```c
#include <stdio.h>
#include <stdbool.h>

// 假设的检测函数
bool check_memory_modification() {
    // 实现检测逻辑
    // 示例
    printf("Performing memory modification check...\n");
    
    // 假设检查了内存，并得出了结果
    bool is_modified = false; // 假设内存没有被修改

    // 返回检测结果
    return is_modified;
}
```

### 编译为共享库

为了编译这段代码为共享库，你需要使用特定的编译器命令。下面提供了在Linux和Windows上编译共享库的示例命令。

#### 在Linux上编译

使用`gcc`编译器：

```sh
gcc -shared -fpic -o libmemory_check.so memory_check.c
```

这会生成一个名为`libmemory_check.so`的共享库。

#### 在Windows上编译

使用`gcc`编译器（例如，MinGW或Cygwin）：

```sh
gcc -shared -o memory_check.dll memory_check.c
```

这会生成一个名为`memory_check.dll`的共享库。

### 使用Node.js调用

编译了C代码为共享库，就可以按照之前提供的Node.js示例来调用`check_memory_modification`函数了。确保更新Node.js代码中的库路径，使其指向你的`.dll`或`.so`文件。

## Node.js 通信部分

为了实现一个具有通信功能的系统，使得Node.js后端可以接收前端请求并返回内存检测的结果，我们需要设置一个简单的HTTP服务器。这个服务器将监听来自前端的请求，执行内存检测，然后将结果以JSON格式发送回前端。

我们将使用Node.js的`http`模块来创建这个服务器，并使用前面提到的`ffi-napi`库来调用C语言编写的`check_memory_modification`函数。

### Node.js HTTP服务器

下面的Node.js代码示例创建了一个HTTP服务器，该服务器响应对`/check-memory`路径的GET请求，并调用之前提到的C函数来检测内存是否被修改。

```javascript
const http = require('http');
const ffi = require('ffi-napi');

// 加载C语言库
let libraryPath = './libmemory_check.so'; // 或 './memory_check.dll' 在Windows上
let memoryCheckLib = ffi.Library(libraryPath, {
  'check_memory_modification': ['bool', []],
});

// 创建HTTP服务器
const server = http.createServer((req, res) => {
  if (req.url === '/check-memory' && req.method === 'GET') {
    // 调用C语言函数检测内存修改
    let isModified = memoryCheckLib.check_memory_modification();
    let response = {
      memoryModified: isModified,
      message: isModified ? "Memory has been modified." : "Memory is not modified."
    };

    // 设置响应头
    res.setHeader('Content-Type', 'application/json');
    // 允许跨域请求
    res.setHeader('Access-Control-Allow-Origin', '*');

    // 发送响应数据
    res.end(JSON.stringify(response));
  } else {
    // 处理其他请求
    res.writeHead(404);
    res.end(JSON.stringify({ message: "Resource not found" }));
  }
});

// 监听端口
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server is listening on port ${PORT}`);
});
```

### 预留通讯接口

上述服务器代码中的`/check-memory`路径就是我们为内存检测功能预留的通讯接口。前端可以通过发送GET请求到这个路径来触发内存检测过程，并获取一个包含检测结果的JSON响应。

为了在实际应用中使用这个接口，前端开发者需要根据自己的需求，使用JavaScript的`fetch`API或者其他HTTP客户端库（如Axios）来向这个接口发送请求，并处理返回的数据。

例如，使用`fetch`API的简单示例可能如下所示：

```javascript
fetch('http://localhost:3000/check-memory')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

这段代码发送一个GET请求到我们的Node.js服务器，并打印出内存检测的结果。

通过这种方式，我们就建立了一个简单的前后端通信机制，允许前端应用查询后端的内存检测服务，并获取结果。
