# vcpkg 使用说明

#### 1. vcpkg 是一个跨平台的c++包管理器，类似于nuget。使用vcpkg可以方便的添加各种开源库

#### 2. 准备环境

1. git

2. vs2015 或更新版本

3. window7 或更新版本

4. 确认安装了Visual Studio 的C++桌面开发环境（这两项一定要勾选！  windows 10 SDK 版本可以不一样）

   ![Configured C++ features](https://i.loli.net/2021/01/28/D9lWLzVoa5fZrqp.png)

#### 3. 开始安装

 1. 使用 **管理员权限** 打开一个terminal （一定要用管理员权限！！）

 2. 选择任意不需要管理员权限的路径进行git clone,这里用 D:/ 为例

    ````bash
    git clone https://github.com/microsoft/vcpkg
    ````

	3. 进入目录并运行安装脚本

    ``` shell
    cd d:/vcpkg
    ./bootstrap-vcpkg.bat
    ./vcpkg.exe integrate install
    ```

	4. （可选）安装完成后，命令行会提示一行配置代码，例如`-DCMAKE_TOOLCHAIN_FILE=D:/CodeEnvironment/vcpkg/scripts/buildsystems/vcpkg.cmake` ，如果使用CMake，可以复制这行使用

#### 4. 使用vcpkg

搜索要安装的库

> `d:/vcpkg/vcpkg.exe search <库名>`

安装指定库

>  `d:/vcpkg/vcpkg.exe install <库名>:<平台版本>`  
>
> 例 :  `.\vcpkg.exe install eigen3:x64-windows` 安装x64-windows版本的eigen3库

在vscode 中使用vcpkg

> 在setting.json 中添加
>
> ```json
> "cmake.configureSettings": {
>           "CMAKE_TOOLCHAIN_FILE": "D:/CodeEnvironment/vcpkg/scripts/buildsystems/vcpkg.cmake"
> }
> ```