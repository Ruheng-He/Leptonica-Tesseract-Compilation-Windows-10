# Windows 10 下 Leptonica、Tesseract 的编译

## 环境

Leptonica `1.78.0`

Tesseract `4.1.1`

Visual Studio `16.11.21`

Visual C++ `2019 14.32.31332.0`

CMake `3.25.1`

Software Network `26-Nov-2022 17:19`

vcpkg `2022-11-10-5fdee72bc1fceca198fb1ab7589837206a8b81ba`

## 编译过程

### 文件准备

**各内容的下载链接**

- leptonica-1.78.0.tar.gz

    `https://github.com/DanBloomberg/leptonica/releases/download/1.78.0/leptonica-1.78.0.tar.gz`

- tesseract-4.1.1.tar.gz

    `https://github.com/tesseract-ocr/tesseract/archive/refs/tags/4.1.1.tar.gz`

- cmake-3.25.1-windows-x86_64.msi

    `https://github.com/Kitware/CMake/releases/download/v3.25.1/cmake-3.25.1-windows-x86_64.msi`

- sw-master-windows-client.zip

    `https://software-network.org/client/sw-master-windows-client.zip`

- vcpkg.exe

    `https://github.com/microsoft/vcpkg/archive/refs/heads/master.zip`

- eng.traineddata

    `https://github.com/tesseract-ocr/tessdata/blob/main/eng.traineddata`

### Software Network 配置

在系统变量 Path 中添加 sw.exe 的路径，以管理员身份运行 Windows PowerShell。

```sh
sw setup # 如果连接了代理服务器，可以尝试 sw --ignore-ssl-checks setup。
```

`setup` 后，相关文件会被放在 `C:\Users\Your_User_Name\\.sw`。

### vcpkg 配置

在 vcpkg 文件夹下以管理员身份运行 Windows PowerShell。

执行命令`.\bootstrap-vcpkg.bat`以产生`vcpkg.exe`。

执行命令`.\vcpkg install package_name --triplet=x64-windows`以获取以下依赖。

tiff `4.4.0#1`

giflib `5.2.1#3`

openjpeg `2.5.0`

ijg-libjpeg `9e`

libpng `1.6.39`

libwebp `1.2.4`

liblzma `5.2.5#6`

pkgconf `1.8.0#3`

vcpkg-pkgconfig-get-modules `2022-02-10#1`

zlib `1.2.13`

举例：`.\vcpkg install tiff --triplet=x64-windows`。

依赖会被存放在`.\vcpkg\packages`。

### 编译Leptonica

运行cmake-gui。

在`Where is the source code`处选择文件夹`leptonica-1.78.0`。

新建一个文件夹，假设名为`build`。

在`Where to build the binaries`处选择文件夹`build`。

点击左下方`Configure`，在弹出窗口的`Specify the generator for this project`处选择`Visual Studio 16 2019`。

在窗口下方选择`Use default native compliers`。

点击Finish后CMake会自动开始配置。

下方log框出现Configuring done后，上方的变量框会出现红色，再次点击Configure，红色消失，Configuring done，log框出现以下缺少相关依赖的提示信息。

```
Could NOT find GIF (missing: GIF_LIBRARY GIF_INCLUDE_DIR) 
Could NOT find JPEG (missing: JPEG_LIBRARY JPEG_INCLUDE_DIR) 
Could NOT find ZLIB (missing: ZLIB_LIBRARY ZLIB_INCLUDE_DIR) 
Could NOT find PNG (missing: PNG_LIBRARY PNG_PNG_INCLUDE_DIR) 
Could NOT find TIFF (missing: TIFF_LIBRARY TIFF_INCLUDE_DIR) 
Could NOT find ZLIB (missing: ZLIB_LIBRARY ZLIB_INCLUDE_DIR) 
Could NOT find PkgConfig (missing: PKG_CONFIG_EXECUTABLE) 
```

在变量框上方勾选Advanced，可找到这些missing的变量。

对显示NOTFOUND的变量值进行补充。

举例如下。

ZLIB_INCLUDE_DIR的值修改为`.\vcpkg\packages\zlib_x64-windows\include`

ZLIB_LIBRARY_DUBUG的值修改为`.\vcpkg\packages\zlib_x64-windows\debug\lib\zlib.lib`

ZLIB_LIBRARY_RELEASE的值修改为`.\vcpkg\packages\zlib_x64-windows\lib\zlib.lib`

PKG_CONFIG_EXECUTABLE的值修改为`D:\vcpkg\packages\pkgconf_x64-windows\tools\pkgconf\pkg.conf.exe`

CMAKE_INSTALL_PREFIX这个值是后面VS编译文件的输出目录，默认在`C:\Program Files (x86)`下，建议不要修改，否则可能影响Tesseract的编译。

再次点击Configure，log框输出以下内容。

```
Looking for WEBP
Found WEBP library
Found openjpeg library
```

配置完成，点击Genrate生成VS工程文件。

打开上文提到的build文件夹，以管理员身份打开后缀为`sln`的文件。

在生成中打开配置管理器，活动解决方案配置选择为Release，活动解决方案平台选择为x64，在项目上下文的生成处勾选`ALL_BUILD` `INSTALL` `leptonica` `ZERO_CHECK`（默认没有勾选`INSTALL`），关闭配置管理器，在生成中选择重新生成解决方案，可在`C:\Program Files (x86)\leptonica`处找到编译好的库文件。正常来说，这个目录的结构应该如下所示。

```
├─bin
├─cmake
├─include
--leptonica
├─lib
--pkgconfig
```

### 编译Tesseract

运行cmake-gui。

在`Where is the source code`处选择文件夹`tesseract-4.1.1`。

新建一个文件夹，假设名为`build_t`。

在`Where to build the binaries`处选择文件夹`build_t`。

点击左下方`Configure`，弹出一个窗口。

在`Specify the generator for this project`处选择`Visual Studio 16 2019`。

在窗口下方选择`Use default native compliers`。

点击`Finish`后CMake会自动开始配置。

下方log框出现`Configuring incomplete, errors occurred!`后，上方的变量框出现红色，再次点击`Configure`，红色消失。

然后取消勾选变量框中的`BUILD_TRAINING_TOOLS`（勾选也行，自行根据报错信息添加相关依赖即可）。

将变量`Leptonica_DIR`的值设置为上文提到的Leptonica的编译好的库文件所在目录的cmake子目录。

`CMAKE_INSTALL_PREFIX`的作用同上。

再次点击`Configure`。

待配置完成，点击`Genrate`生成VS解决方案文件。

打开上文提到的`build_t`文件夹，以管理员身份打开后缀为`sln`的文件。

在生成中打开配置管理器，将活动解决方案配置选择为`Release`，将活动解决方案平台选择为x64，在项目上下文的生成处勾选`ALL_BUILD` `INSTALL` `libtesseract` `tesseract` `ZERO_CHECK`（默认没有勾选`INSTALL`），关闭配置管理器，在生成中选择重新生成解决方案，即可在上文的`CMAKE_INSTALL_PREFIX`这个目录找到编译好的库文件。正常来说，这个目录的结构应该如下所示。

```
├─bin
├─cmake
├─include
--tesseract
├─lib
--pkgconfig
```

## 测试

现在`C:\Program Files (x86)`下的`leptonica`和`tesseract`文件夹可以移动。

在VS中新建一个C++空项目。

打开`项目-->属性-->C/C++-->常规-->附加包含目录`，包含`leptonica`和`tesseract`头文件所在的目录，如下所示。

```
.\tesseract\include\tesseract
.\leptonica\include\leptonica
```

打开`项目-->属性-->链接器-->常规-->附加库目录`，包含`leptonica`和`tesseract`头文件所在的目录，如下所示。

```
.\leptonica\lib
.\tesseract\lib
```

打开项目-->属性-->链接器-->输入-->附加依赖项，输入leptonica和tesseract库文件的名字，如下所示。

```
tesseract41.lib
leptonica-1.78.0.lib
```

点击应用、确定。

测试如下代码，如无意外的话，应该会缺少某些dll。在`.\vcpkg\packages`下可以找到这些dll文件，复制到程序所在文件夹即可。测试时还缺少一个`jpeg62.dll`，想办法搞到它。指定英语语言数据eng.traineddata和测试图片的路径，运行程序，输出图片的OCR结果。

```c
#include "baseapi.h" 
#include "allheaders.h"
#include <iostream>

int main()
{
    char * outText;

    tesseract::TessBaseAPI * api = new tesseract::TessBaseAPI();

    if(api->Init("c:/", "eng")){

        fprintf(stderr, "Could not initialize tesseract.\n");

        exit(1);

    }

    Pix* image = pixRead("C:/test.jpg");

    api->SetImage(image);

    outText = api->GetUTF8Text();

    printf("OCR output:\n%s", outText);

    api->End();

    delete api;

    delete[] outText;

    pixDestroy(&image);

    return 0;

}
```

完成。
