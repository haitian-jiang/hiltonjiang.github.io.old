---
title: use torch on mac
category: foobar
date: 2022-6-6
---



### How to run torch in C++ on M1 mac

This article records the steps for installing, configuring and running `pytorch` and torch library in C++ natively on M1 mac.

##### Step 1: Install `pytorch` from conda

I use this command to install `pytorch` from conda-forge source. Back then there was no officially compiled arm version `pytorch` for mac. 

```bash
conda install pytorch -c pytorch '-c=conda-forge'
```

Now you can use the CPU version `pytorch` natively on your arm mac.

The dynamic link libraries of `torch` is in `{your/conda/environ}/lib/python3.x/site-packages/torch/lib/*.dylib`. For me, it is like  `/opt/miniconda3/lib/python3.8/site-packages/torch/lib/libtorch.dylib`.



##### Step 2: Get the dependencies ready

- "protobuf" is required by `Caffe2`, you can install it by`brew install protobuf`. If you can use the `protoc` command, then it is successfully installed.

- Compile `kineto`. It is required by `caffe`. 

  ```bash
  git clone --recursive https://github.com/pytorch/kineto.git
  cd kineto/libkineto
  mkdir build && cd build
  cmake ..
  make
  cp libkineto.a ~/.local/bin  # make sure it is in $PATH
  ```



##### Step 3: Use `CMakeList.txt` to build

You can use CLion, which employs `cmake` in its build procedure. 

```cmake
cmake_minimum_required(VERSION 3.20)
project(awesomeProject)

set(CMAKE_CXX_STANDARD 14)

# add torch into cmake path
execute_process(
        COMMAND python -c "import torch; import os; print(os.path.dirname(torch.__file__), end='')"
        OUTPUT_VARIABLE TorchPath
)
list(APPEND CMAKE_PREFIX_PATH ${TorchPath})

# show torch version
execute_process(
        COMMAND python -c "import torch; print(torch.__version__, end='')"
        OUTPUT_VARIABLE TorchVersion
)
message(STATUS "Torch Version: ${TorchVersion}")

include_directories(${TORCH_INCLUDE_DIRS})

find_package(Torch REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

add_executable(awesomeProject main.cpp)
target_link_libraries(awesomeProject "${TORCH_LIBRARIES}")
```



##### Step 3.5: Use `cling` with `torch`

```c++
$ cling -std=c++14
#pragma cling add_include_path("/opt/miniconda3/lib/python3.8/site-packages/torch/include")
#pragma cling add_include_path("/opt/miniconda3/lib/python3.8/site-packages/torch/include/torch/csrc/api/include")
#pragma cling add_library_path("/opt/miniconda3/lib/python3.8/site-packages/torch/lib")
#pragma cling load("libtorch")
#pragma cling load("libtorch_cpu")
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
#include <torch/torch.h>
#include <iostream>
torch::Tensor tensor = torch::eye(3);
```

```c++
#pragma cling add_include_path("/opt/miniconda3/envs/spark/lib/python3.8/site-packages/torch/include")
#pragma cling add_include_path("/opt/miniconda3/envs/spark/lib/python3.8/site-packages/torch/include/torch/csrc/api/include")
#pragma cling add_library_path("/opt/miniconda3/envs/spark/lib/python3.8/site-packages/torch/lib")
#pragma cling load("libtorch")
#pragma cling load("libtorch_cpu")
#pragma cling load("libtorch_global_deps.dylib")
#include <torch/torch.h>
#include <iostream>
//torch::Tensor tensor = torch::eye(3);
//std::cout << tensor << std::endl;
torch::TensorOptions opt = torch::TensorOptions().dtype(torch::kInt32).device(torch::kMPS);
torch::Tensor t = torch::eye(3, opt);
std::cout << t << std::endl;

```

