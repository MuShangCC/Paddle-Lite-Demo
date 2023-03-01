# 目标检测 C++ API Demo 使用指南
在 IOS 上实现实时的目标检测功能，此 Demo 有很好的的易用性和开放性，如在 Demo 中跑自己训练好的模型等。
本文主要介绍目标检测 Demo 运行方法和如何在更新模型/输入/输出处理下，保证目标检测 Demo 仍可继续运行。

## 如何运行目标检测 Demo

### 环境准备

1. 在本地环境安装好 Xcode 工具，详细安装方法请见[Xcode 官网](https://developer.apple.com/cn/xcode/resources/)。
2. 准备一部 Iphone 手机，并在 Xcode 中连接自己的手机 （第一次连接 IPhone 到电脑时，需要在 IPhone 的 `设置->通用->设备管理` 中选择本电脑并信任）

<p align="center">
<src="https://paddlelite-demo.bj.bcebos.com/demo/object_detection/docs_img/ios/Xcode-phone.jpg"/>
</p>

### 部署步骤

1. 目标检测 Demo 位于 `Paddle-Lite-Demo/object_detection/ios/ssd_mobilnetv1_demo`  目录
2.  cd `Paddle-Lite-Demo/libs` 目录，运行 `download.sh` 脚本，下载所需要的 Paddle Lite 预测库
3.  cd `Paddle-Lite-Demo/object_detection/assets` 目录，运行 `download.sh` 脚本，下载 OPT 优化后模型

```shell
cd Paddle-Lite-Demo/libs
# 下载所需要的 Paddle Lite 预测库
sh download.sh
cd ../object_detection/assets
# 下载OPT 优化后模型
sh download.sh
cd ..
```

4.  用 Xcode 打开  `ssd_mobilnetv1_demo/ssd_mobilnetv1_demo.xcodeproj`  文件，修改工程配置。依次修改  `General/Identity`  和 `Signing&Capabilities`  属性，替换为自己的工程代号和团队名称。（必须修改，不然无法通过编译）

    <p align="center">
    <src="https://paddlelite-demo.bj.bcebos.com/demo/object_detection/docs_img/ios/Xcode1.png"/>
    </p>

    <p align="center">
    <src="https://paddlelite-demo.bj.bcebos.com/demo/object_detection/docs_img/ios/Xcode2.png"/>
    </p>


5.  选中 `ssd_mobilnetv1_demo/third-party` 目录 ，右击选择 `Add Files to "third-party" ...`  选项，将预测库、Opencv库和 assets内容（模型、测试图片及标签文件）添加到工程中。
      
**注意：**
  如果觉得上述方法比较麻烦，可以使用工程下的 `prepare.sh` 脚本，完成上述资源的拷贝
        
  ```shell
    # path = Paddle-Lite-Demo/ios/object_detection
    sh prepare.sh
  ```
        
6.  IPhone 手机连接电脑，在 Xcode 中连接自己的手机 （第一次连接 IPhone 到电脑时，需要在 IPhone 的 `设置->通用->设备管理` 中选择本电脑并信任）

<p align="center">
<img width="300" height="350"  src="https://paddlelite-demo.bj.bcebos.com/demo/object_detection/docs_img/ios/Xcode-phone.jpg"/>
</p>


7. 按下左上角的 Run 按钮，自动编译 APP 并安装到手机。在苹果手机中设置信任该 APP（进入 `设置->通用->设备管理`，选中新安装的 APP 并 `验证该应用`）
成功后效果如下：

  | APP 图标 | APP 效果 |
  | ---     | --- |
  | ![app_pic](https://paddlelite-demo.bj.bcebos.com/demo/object_detection/docs_img/ios/IOS_app.jpeg)    | ![app_res](https://paddlelite-demo.bj.bcebos.com/demo/object_detection/docs_img/ios/app_run_res.jpg) |

>> 备注：本 demo 支持 Picodet 增强版模型，即将模型的后处理写入网络中，用户无需书写复杂的后处理nms等逻辑操作。

|序号| 说明 |[Picodet]( https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.4/configs/picodet/legacy_model)和	[Picodet 增强版]( https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.4/configs/picodet)|	
|---|---|---|
|0|算法前后处理|Picodet 和 Picodet 增强版，算法的前、后处理完全相同<br>1.前处理：包括 Detection 常见的减均值等操作<br>2.后处理：包括 nms 等|
|1|后处理写入模型结构|1. 该 Picodet Demo 使用[PicoDet-S 320*320](https://paddledet.bj.bcebos.com/deploy/Inference/picodet_s_320_coco_lcnet.tar)实现<br>2.该 Demo 仅支持增强版模型替换，即将模型的后处理写入网络，具体操作方法请参考[PaddleDetection导出部分](https://github.com/PaddlePaddle/PaddleDetection/tree/release/2.4/configs/picodet#%E5%AF%BC%E5%87%BA%E5%8F%8A%E8%BD%AC%E6%8D%A2%E6%A8%A1%E5%9E%8B)）<br> **注意：**替换模型时，需要修改输入shape、类别数等；（1）详细修改数值需结合实际训练情况完成（2）修改代码位置下方文档说明|
|2|后处理不在模型结构|该 Demo无 后处理解码+nms逻辑，不支持「不带后处理的模型」替换工作，需要用户自行添加完成|

## 更新预测库

*Paddle Lite 项目：https://github.com/PaddlePaddle/Paddle-Lite
* 参考 [Paddle Lite 源码编译文档](https://www.paddlepaddle.org.cn/lite/develop/source_compile/compile_env.html)，编译 iOS 预测库
* 编译最终产物位于 `build.lite.xxx.xxx.xxx` 下的 `inference_lite_lib.xxx.xxx`
  * 替换 c++ 库
       * 头文件
         将生成的 `build.lite.ios.xxx.clang/inference_lite_lib.ios64.xxx/include` 文件夹替换 Demo 中的 `Paddle-Lite-Demo/object_detection/ios/ssd_mobilnetv1_demo/detection_demo/third-party/PaddleLite/include`
       * 替换 arm64-v8a 库
         将生成的 `build.lite.ios.ios64.armv8/inference_lite_lib.ios64.armv8/libs/libpaddle_api_light_bundled.a` 库替换 Demo 中的 `Paddle-Lite-Demo/object_detection/ios/ssd_mobilnetv1_demo/detection_demo/third-party/PaddleLite/lib/libpaddle_api_light_bundled.a`

>**注意：**
>> 如果要使用 armv7 库，则可将 armv7 库替换至相应目录下：
>> * armeabi-v7a
>>  将生成的 `build.lite.ios.ios.armv7/inference_lite_lib.ios.armv7/libs/libpaddle_api_light_bundled.a` 库替换 Demo 中的 `Paddle-Lite-Demo/object_detection/ios/ssd_mobilnetv1_demo/detection_demo/third-party/PaddleLite/lib/libpaddle_api_light_bundled.a`
  
  
## Demo 内容介绍

先整体介绍下目标检测 Demo 的代码结构，然后再介绍 Demo 每部分功能.

### 整体结构介绍
1.  `third-party`： 存放预测库、模型、测试图片等相关信息
      * `assets`: 存放预测资源
        - models：模型文件，opt 工具转化后 Paddle Lite 模型
        - images：测试图片
        - labels：标签文件
      * `PaddleLite`：存放 Paddle Lite 预测库和头文件
        - lib
        - include
      * `opencv2.framework`：opencv  库和头文件

    ```shell
    # 位置：
    detection_demo/third-party/
    example：
    # IOS 预测库
    detection_demo/third-party/PaddleLite/lib/libpaddle_api_light_bundled.a
    # 预测库头文件
    detection_demo/third-party/PaddleLite/include/paddle_api.h
    detection_demo/third-party/PaddleLite/include/paddle_use_kernels.h
    detection_demo/third-party/PaddleLite/include/paddle_use_ops.h
    ```

 2.  `ViewController.mm`：主要预测代码

    ```shell
    # 位置
    detection_demo/ViewController.mm
    ``` 

### `ViewController.mm`  文件内容介绍

 * `viewDidLoad`  方法
    APP 界面初始化、推理引擎 predictor 创建和运行方法，这个方法包含界面参数获取、predictor 构建和运行、图像前/后处理等内容
   
 * `processImage` 方法
   实现图像输入变化时，进行新的推理，并获取相应的输出结果

* `pre_process` 方法
   输入预处理操作

* `post_process` 方法
   输出后处理操作中

## 代码讲解 （使用 Paddle Lite `C++ API` 执行预测）

IOS 示例基于 C++ API 开发，调用 Paddle Lite `C++s API` 包括以下五步。更详细的 `API` 描述参考：[Paddle Lite C++ API ](https://www.paddlepaddle.org.cn/lite/develop/api_reference/cxx_api_doc.html)。

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
```

## 如何更新模型和输入/输出预处理

### 更新模型
1. 将优化后的模型存放到目录 `third-party/assets/models/` 下；
2. 如果模型名字跟工程中模型名字一模一样，即均是使用 `third-party/assets/models/ssd_mobilenet_v1_pascalvoc_for_cpu/model.nb`，则代码不需更新；否则话，需要修改 `./ViewController.mm` 中代码

  以更新 ssd_mobilenet_v3 模型为例，则先将优化后的模型存放到 `third-party/assets/models/ssd_mobilenet_v3_for_cpu/mv2.nb` 下，然后更新代码

```c++
// 代码文件 `image_classifictaion/ViewController.mm`
- (void)viewDidLoad {
...
MobileConfig config;
// old
// config.set_model_from_file(app_dir+ "/models/mobilenet_v1_for_cpu/model.nb");
// update now
config.set_model_from_file(app_dir+ "/models/ssd_mobilenet_v3_for_cpu/mv2.nb");
predictor = CreatePaddlePredictor<MobileConfig>(config);
...
}
```

**注意：**

 - 如果更新后模型的输入信息如Shape、Tensor个数等发生改变，需要更新 `ViewController.mm` 文件中 `pre_process(...)` 输入预处理方法，完成模型输入更新
 - 如果更新后模型的输出信息发生改变，需要更新 `ViewController.mm` 文件中 `post_process(...)` 输出后处理方法，完成模型输出更新即可
 
- 如果需要更新 `label.txt`，则需将更新后的标签文件，存放至`third-party/assets/labels/` 目录下。
若更新后标签名字不一样，应修改代码文件 `./ViewController.mm` 中代码 

```c++
// 代码文件 `image_classifictaion/ViewController.mm`
- (void)viewDidLoad {
...
// old
// std::string label_file_str = app_dir+"/labels/pascalvoc_label_list";
// update now
std::string label_file_str = app_dir+"/labels/labels_new.txt";
self.labels = [self load_labels:label_file_str];
...
}
```

### 更新输入/输出预处理
1. 更新输入数据

- 将更新的图片存放在 `third-party/assets/images/` 下；
- 更新文件 `detection_demo/ViewController.mm` 中的代码

以更新 `cat.jpg` 为例，则先将 `cat.jpg` 存放在 `third-party/assets/images/` 下，然后更新代码

```c++
// 代码文件 `image_classifictaion/ViewController.mm`
- (void)viewDidLoad {
...
// old
// _image = [UIImage imageNamed:@"third-party/assets/images/dog.jpg"];
// now
_image = [UIImage imageNamed:@"third-party/assets/images/cat.jpg"];
if (_image != nil) {
    printf("load image successed\n");
    imageView.image = _image;
} else {
    printf("load image failed\n");
}
...
}
```

**注意：** 本 Demo 是支持图片/视频流/拍照三种输入方式，如果需更新输入图片建议通过 APP 的拍照或视频流方式进行更新，这样不用修改代码，则能正常推理。


2. 更新输入预处理
此处需要更新 `image_classifictaion/ViewController.mm` 中的 `pre_process(...)` 输入预处理方法

3. 更新输出预处理
此处需要更新 `image_classifictaion/ViewController.mm` 中的 `post_process(...)` 输出后处理方法

### 其他文件
* `time.h` 包含常见的计时处理函数，用于计时处理

## 性能优化方法
如果你觉得当前性能不符合需求，想进一步提升模型性能，可参考[首页中性能优化文档](/README.md)完成性能优化。
