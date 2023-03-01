# 人像分割 C++ API Demo 使用指南

在 Android 上实现人像分割功能，此 Demo 有很好的的易用性和开放性，如在 Demo 中跑自己训练好的模型等.
本文主要介绍人像分割 Demo 运行方法和如何在更新模型/输入/输出处理下，保证人像分割 demo 仍可继续运行。

## 如何运行人像分割 Demo

### 环境准备

1. 在本地环境安装好 Android Studio 工具，详细安装方法请见[Android Stuido 官网](https://developer.android.com/studio)。
2. 准备一部 Android 手机，并开启 USB 调试模式。开启方法: `手机设置 -> 查找开发者选项 -> 打开开发者选项和 USB 调试模式`

**注意**:
> 如果您的 Android Studio 尚未配置 NDK ，请根据 Android Studio 用户指南中的[安装及配置 NDK 和 CMake ](https://developer.android.com/studio/projects/install-ndk)内容，预先配置好 NDK 。您可以选择最新的 NDK 版本，或者使用。
Paddle Lite 预测库版本一样的 NDK

### 部署步骤

1. 人像分割 Demo 位于 `Paddle-Lite-Demo/human_segmentation/android/app/java/human_segmentation` 目录
2. 用 Android Studio 打开 human_segmentation 工程
3. 手机连接电脑，打开 USB 调试和文件传输模式，并在 Android Studio 上连接自己的手机设备（手机需要开启允许从 USB 安装软件权限）

**注意**:
> 如果您在导入项目、编译或者运行过程中遇到 NDK 配置错误的提示，请打开 ` File > Project Structure > SDK Location`，修改 `Andriod NDK location` 为您本机配置的 NDK 所在路径。
> 如果您是通过 Andriod Studio 的 SDK Tools 下载的 NDK (见本章节"环境准备")，可以直接点击下拉框选择默认路径。
> 还有一种 NDK 配置方法，你可以在 `human_segmentation/local.properties` 文件中手动添加 NDK 路径配置 `nkd.dir=/root/android-ndk-r20b`
> 如果以上步骤仍旧无法解决 NDK 配置错误，请尝试根据 Andriod Studio 官方文档中的[更新 Android Gradle 插件](https://developer.android.com/studio/releases/gradle-plugin?hl=zh-cn#updating-plugin)章节，尝试更新Android Gradle plugin版本。

4. 点击 Run 按钮，自动编译 APP 并安装到手机。(该过程会自动下载 Paddle Lite 预测库和模型，需要联网)
成功后效果如下，图一：APP 安装到手机        图二： APP 打开后的效果，会自动分割人像

  | APP 图标 | APP 效果 |
  | ---     | --- |
  | ![app_pic](https://paddlelite-demo.bj.bcebos.com/demo/human_segmentation/doc_images/android/install_app.jpg)    | ![app_res](https://paddlelite-demo.bj.bcebos.com/demo/human_segmentation/doc_images/android/process_success.jpg) |

## 更新预测库

* Paddle Lite 项目：https://github.com/PaddlePaddle/Paddle-Lite
 * 参考 [Paddle Lite 源码编译文档](https://www.paddlepaddle.org.cn/lite/develop/source_compile/compile_env.html)，编译 Android 预测库
 * 编译最终产物位于 `build.lite.xxx.xxx.xxx` 下的 `inference_lite_lib.xxx.xxx`
    * 替换 java 库
        * jar 包
          将生成的 `build.lite.android.xxx.clang/inference_lite_lib.android.xxx/java/jar/PaddlePredictor.jar` 替换 Demo 中的 `Paddle-Lite-Demo/human_segmentation/android/app/java/human_segmentation/app/libs/PaddlePredictor.jar`
        * Java so
            * armeabi-v7a
              将生成的 `build.lite.android.armv7.clang/inference_lite_lib.android.armv7/java/so/libpaddle_lite_jni.so` 库替换 Demo 中的 `Paddle-Lite-Demo/human_segmentation/android/java/human_segmentation/app/src/main/jniLibs/armeabi-v7a/libpaddle_lite_jni.so`
            * arm64-v8a
              将生成的 `build.lite.android.armv8.clang/inference_lite_lib.android.armv8/java/so/libpaddle_lite_jni.so` 库替换 Demo 中的 `Paddle-Lite-Demo/human_segmentation/android/java/human_segmentation/app/src/main/jniLibs/arm64-v8a/libpaddle_lite_jni.so`              

## Demo 内容介绍

先整体介绍下目标检测 Demo 的代码结构，然后介绍 Java 各功能模块的功能。

### 重点关注内容

1. `Native.java`： Java 预测代码

```shell
# 位置：
Segmentation/app/src/main/java/com/baidu/paddle/lite/demo/segmentation/Native.java
```

2. `Native.cc`： Jni 预测代码用于 Java 与 C++ 语言传递信息

```shell
# 位置：
Segmentation/app/src/main/cpp/Native.cc
```

3. `Pipeline.cc`： C++ 预测代码

```shell
# 位置：
Segmentation/app/src/main/cpp/Pipeline.cc
```

4. `model.nb` : 模型文件 (opt 工具转化后 Paddle Lite 模型), `labels.txt`：训练模型时的 `labels` 文件

```shell
# 位置：
Segmentation/app/src/main/assets/models/model.nb
Segmentation/app/src/main/assets/labels/labels.txt
```

5. `libpaddle_lite_api_shared.so`：Paddle Lite C++ 预测库

```shell
# 位置
Segmentation/app/PaddleLite/cxx/libs/arm64-v8a/libpaddle_lite_api_shared.so
# 如果要替换动态库 so，则将新的动态库 so 更新到此目录下
```

6. `build.gradle` : 定义编译过程的 gradle 脚本。（不用改动，定义了自动下载 Paddle Lite 预测和模型的过程）

```shell
# 位置
Segmentation/app/build.gradle
# 如果需要手动更新模型和预测库，则可将 gradle 脚本中的 `download*` 接口注释即可
```

7. `CMakeLists.txt` : C++ 预测库代码的编译脚本，用于生成 jni 的动态库 `lib_Native.so`

```shell
# 位置
Segmentation/app/cpp/CMakeLists.txt
# 如果有cmake 编译选项更新，可以在 CMakeLists.txt 进行修改即可
```

### Java 端

* 模型存放，将下载好的模型解压存放在 `app/src/assets/models` 目录下
* common Java 包
    在 `app/src/java/com/baidu/paddle/lite/demo/common` 目录下，实现一些公共处理内容，一般不用修改。其中，Utils.java 用于存放一些公用的且与 Java 基类无关的功能，例如模型拷贝、字符串类型转换等
* Segmentation Java 包
    在 `app/src/java/com/baidu/paddle/lite/demo/segmentation` 目录下，实现 APP 界面消息事件和 Java/C++ 端代码互传的桥梁功能
* MainActivity
    实现 APP 的创建、运行、释放功能
    重点关注 `onLoadModel` 和 `onRunModel` 函数，实现 APP 界面值传递和推理处理
     
    ```java
     public boolean onLoadModel() {
        // push model to sdcard
        String realModelDir = getExternalFilesDir(null) + "/" + modelPath;
        Utils.copyDirectoryFromAssets(this, modelPath, realModelDir);

        // push label to sdcard
        String realLabelPath = getExternalFilesDir(null) + "/" + labelPath;
        Utils.copyFileFromAssets(this, labelPath, realLabelPath);
        return predictor.init(realModelDir, realLabelPath, cpuThreadNum,
                cpuPowerMode,
                inputShape, inputMean,
                inputStd, topK);
    }

    public boolean onRunModel() {
        return predictor.isLoaded() && predictor.process();
    }
    ```
   
* SettingActivity
    实现设置界面各个元素的更新与显示，如果新增/删除界面的某个元素，均在这个类里面实现
      备注：

      - 参数的默认值可在 `app/src/main/res/values/strings.xml` 查看
      - 每个元素的 ID 和 value 是对应 `app/src/main/res/xml/settings.xml` 和 `app/src/main/res/values/string.xml` 文件中的值
      - 这部分内容不建议修改，如果有新增属性，可以按照此格式进行添加

* Native
    实现 Java 与 C++ 端代码互传的桥梁功能
    包含三个功能：`init`初始化、 `process`预测处理 和 `release`释放
    备注：
        Java 的 native 方法和 C++ 的 native 方法要一一对应
      
     ```java
     // 初始化函数
      public boolean init(String modelDir,
                        String labelPath,
                        int cpuThreadNum,
                        String cpuPowerMode,
                        long[] inputShape,
                        float[] inputMean,
                        float[] inputStd,
                        int topk);
      // 释放资源
      public boolean release();
      // 预测处理函数，包含预处理、预测和后处理全流程
      public boolean process();
    ```

### C++ 端（native）

* Native
  实现 Java 与 C++ 端代码互传的桥梁功能，将 Java 数值转换为 c++ 数值，调用 c++ 端完成人像分割功能
  **注意：**
  Native 文件生成方法：
  
  ```shell
    cd app/src/java/com/baidu/paddle/lite/demo/segmentation
    # 在当前目录会生成包含 Native 方法的头文件，用户可以将其内容拷贝至 `cpp/Native.cc` 中
    javac -classpath D:\dev\android-sdk\platforms\android-29\android.jar -encoding utf8 -h . Native.java 
  ```

* Pipeline
  实现输入预处理、推理执行和输出后处理的流水线处理，支持单/多个模型的串行处理

* Utils
  实现其他辅助功能，如 `NHWC` 格式转 `NCHW` 格式、字符串处理等

* 新增模型支持
  - 在 Pipeline 文件中新增模型的预测类，实现图像预处理、预测和图像后处理功能
  - 在 Pipeline 文件中 `Pipeline` 类添加该模型预测类的调用和处理

## 代码讲解 （使用 Paddle Lite `C++ API` 执行预测）

Android 示例基于 C++ API 开发，调用 Paddle Lite `C++s API` 包括以下五步。更详细的 `API` 描述参考：[Paddle Lite C++ API ](https://www.paddlepaddle.org.cn/lite/develop/api_reference/cxx_api_doc.html)。

```c++
#include <iostream>
// 引入 C++ API
#include "include/paddle_api.h"
#include "include/paddle_use_ops.h"
#include "include/paddle_use_kernels.h"

// 1. 设置 MobileConfig
MobileConfig config;
config.set_model_from_file(<modelPath>); // 设置 NaiveBuffer 格式模型路径
config.set_power_mode(LITE_POWER_NO_BIND); // 设置 CPU 运行模式
config.set_threads(4); // 设置工作线程数

// 2. 创建 PaddlePredictor
std::shared_ptr<PaddlePredictor> predictor = CreatePaddlePredictor<MobileConfig>(config);

// 3. 设置输入数据
std::unique_ptr<Tensor> input_tensor(std::move(predictor->GetInput(0)));
input_tensor->Resize({1, 3, 224, 224});
auto* data = input_tensor->mutable_data<float>();
for (int i = 0; i < ShapeProduction(input_tensor->shape()); ++i) {
  data[i] = 1;
}
// 如果输入是图片，则可在第三步时将预处理后的图像数据赋值给输入 Tensor

// 4. 执行预测
predictor->run();

// 5. 获取输出数据
std::unique_ptr<const Tensor> output_tensor(std::move(predictor->GetOutput(0)));
std::cout << "Output shape " << output_tensor->shape()[1] << std::endl;
for (int i = 0; i < ShapeProduction(output_tensor->shape()); i += 100) {
  std::cout << "Output[" << i << "]: " << output_tensor->data<float>()[i]
            << std::endl;
}

// 例如图像分类：输出后处理，输出检测结果=
auto outputData = outputTensor->data<float>();
auto outputShape = outputTensor->shape();
int outputSize = ShapeProduction(outputShape);
for (int i = 0; i < outputSize; i += 6) {
  // Class id
  auto class_id = static_cast<int>(round(outputData[i]));
  // Confidence score
  auto score = outputData[i + 1];
  if (score < scoreThreshold_)
    continue;
  RESULT object;
  object.class_name = class_id >= 0 && class_id < labelList_.size()
                          ? labelList_[class_id]
                          : "Unknow";
  object.fill_color = class_id >= 0 && class_id < colorMap_.size()
                            ? colorMap_[class_id]
                            : cv::Scalar(0, 0, 0);
  object.score = score;
  object.x = MIN(MAX(outputData[i + 2], 0.0f), 1.0f);
  object.y = MIN(MAX(outputData[i + 3], 0.0f), 1.0f);
  object.w = MIN(MAX(outputData[i + 4] - outputData[i + 2], 0.0f), 1.0f);
  object.h = MIN(MAX(outputData[i + 5] - outputData[i + 3], 0.0f), 1.0f);
  results->push_back(object);
}
```

## 如何更新模型和输入/输出预处理

### 更新模型
1. 将优化后的模型存放到目录 `Segmentation/app/src/main/assets/models/` 下；
2. 如果模型名字跟工程中模型名字一模一样，即均是使用 `model.nb`，则代码不需更新；否则话，需要修改 `Segmentation/app/src/main/java/com.baidu.paddle.lite.demo.segmentation/MainActivity.java` 中代码：
例子：假设更新 mobilenet_v2 模型为例，则先将优化后的模型存放到 `Segmentation/app/src/main/assets/models/mobilenet_v2_for_cpu/mv2.nb` 下，然后更新代码

```c++
// 代码文件 `Segmentation/app/src/main/java/com.baidu.paddle.lite.demo.segmentation/MainActivity.java`
public boolean onLoadModel() {
  modelPath = "models/mobilenet_v2_for_cpu/"; // change modelPath
  return predictor.init(MainActivity.this, modelPath, labelPath, cpuThreadNum,
                cpuPowerMode, inputColorFormat, inputShape, inputMean,
                inputStd);
}
```

**注意：**
- 如果优化后的模型名字不是 `model.nb`，则需要将优化后的模型名字更新为 `model.nb` 或修改 `Segmentation/app/src/main/cpp/Pipeline.cc`

- 本 Demo 提供了 setting 界面，可以在将新模型 mobilenet_v2 放在 `assets/models/`后，不用手动更新代码，直接在安装好 APP 的 setting 界面更新模型路径即可

- 如果更新模型的输入/输出 Tensor 个数、shape 和 Dtype 发生更新，需要更新文件 `Segmentation//app/src/main/cpp/Pipeline.cc` 的 `preprocess` 预处理和 `postProcess` 后处理代码即可。

- 如果需要更新标签文件，则需要将新的标签文件存放在目录 `Segmentation/app/src/main/assets/labels/` 下，并更新 `Segmentation/app/src/main/java/com.baidu.paddle.lite.demo.segmentation/MainActivity.java` 中的标签文件路径名。也可直接在 `Pipeline.cc` 中的构造函数中修改模型、标签路径。

```c++
Segmentation::Segmentation(const std::string &modelDir,
                           const std::string &labelPath,
                           const int cpuThreadNum,
                           const std::string &cpuPowerMode,
                           const std::vector<int64_t> &inputShap,
                           const std::vector<float> &inputMean,
                           const std::vector<float> &inputStd)
        : inputShape_(inputShap), inputMean_(inputMean), inputStd_(inputStd) {
  paddle::lite_api::MobileConfig config;
  config.set_model_from_file(modelDir + "/model.nb");
  config.set_threads(cpuThreadNum);
  config.set_power_mode(ParsePowerMode(cpuPowerMode));
  predictor_ =
          paddle::lite_api::CreatePaddlePredictor<
          paddle::lite_api::MobileConfig>(config);
  labelList_ = LoadLabelList(labelPath);
}
```

### 更新输入/输出预处理
1. 更新输入数据

- 将更新的图片存放在 `Segmentation/app/src/main/assets/images/` 下；
- 更新文件 `Segmentation/app/src/main/java/com.baidu.paddle.lite.demo.segmentation/MainActivity.java`  中的代码


**注意**:
> 本 Demo 支持拍照和从相册加载新图片进行推理。此处想更新图片，可通过拍照或从相册加载图片方式实现。

2. 更新输入预处理
此处需要更新 `Segmentation/app/src/main/cpp/Pipeline.cc` 中的`preprocess` 预处理代码实现。

3. 更新输出预处理
此处需要更新 `Segmentation/app/src/main/cpp/Pipeline.cc` 中的 `postProcess` 后处理代码实现。

## 通过 setting 界面更新人像分割的相关参数

### setting 界面参数介绍
可通过APP上的 Settings 按钮，实现人像分割demo中些许参数的更新，目前支持以下参数的更新：
参数的默认值可在 `app/src/main/res/values/strings.xml` 查看
- model setting：（需要提前将模型/图片/标签放在assets 目录，或者通过adb push 将其放置手机目录）
    - model_path 默认是 `models`
    - image_path 默认是 `images/human.jpg`
    - label_path 默认是 `labels/labels.txt`
- CPU setting：
    - power_mode 默认是 `LITE_POWER_HIGH`
    - thread_num 默认是 1
- input setting：
    -input_shape 默认是 `1, 3, 513, 513`
    - input_image_format 默认是 `RGBA`

### setting 界面参数更新
1）打开 APP，点击右上角的 `:` 符合，选择 `Settings..` 选项，打开 setting 界面；
<p align="center">
<img width="250" height="500" src="https://paddlelite-demo.bj.bcebos.com/demo/human_segmentation/doc_images/android/param_setting.jpg"/>
</p>

2）再将 setting 界面的 Enable custom settings 选中☑️，然后更新部分参数；
<p align="center">
<img width="250" height="500" src="https://paddlelite-demo.bj.bcebos.com/demo/human_segmentation/doc_images/android/setting.jpg"/>
</p>

3）假设更新线程数据，将 CPU Thread Num 设置为 4，更新后，返回原界面，APP将自动重新预测，并打印 4 线程的耗时和结果
<p align="center">
<img width="250" height="500" src="https://paddlelite-demo.bj.bcebos.com/demo/human_segmentation/doc_images/android/cpu_threads.jpg"/>
</p>

## 性能优化方法
如果你觉得当前性能不符合需求，想进一步提升模型性能，可参考[首页中性能优化文档](/README.md)完成性能优化。
