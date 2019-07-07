# NModAPI 文档

## 其它语言
- [English](README.md)
- [Pусский](README-RU.md)
- [简体中文](README-zh-CN.md) （当前的语言）

## NMod 是什么？
NMod 的全称是 *Native-Mod* 。*Native-Mod* 通过原生代码（C/C++）来修改 *Minecraft* ，所以被称为 NMod 。

众所周知，MCPE 主要由 C++ 编写。NMod 比 ModPE 脚本有着更好的修改效果，为什么不学着去做 NMod 呢？

但是，制作 NMod 不是一件易事，因为 C++ 语言比 JavaScript 更难，以及原生修改方法的操作不是那么容易理解的。以下的内容将告诉你如何开发 NMod 。

## 开发准备
1. C++ 基础
2. 一个支持 NDK 的 Android IDE（如 Android Studio, Eclipse, AIDE 等）
3. 知道如何构建 Android 原生库（`*.so`）
4. Json 语法

## 链接库
NMod 通过链接 `libsubstrate.so` 和 `libminecraftpe.so` 以执行对 MCPE 的修改。请确保你的 NMod 已链接至 *substrate* 模块与 *minecraftpe* 模块（你可以通过阅读我们的实例知悉如何链接这些库）。

## 事件监听器
NModAPI 为 **NMod 加载完毕、游戏启动与游戏退出** 三个事件提供了监听器，你只需定义这些方法。当事件发生时，NModAPI 将会调用这些方法。

### NMod 加载完毕（OnLoad）
  
```cpp
NMod_OnLoad(JavaVM* javaVM, JNIEnv* jniEnv, const char* minecraftVersion, const char* nmodApiVersion, const char* pathOflibminecraftpeso)
```

- `javaVM` 是一个指向 `JavaVM` 的指针。
- `jniEnv` 是一个指向 `JNIEnv` 的指针。
- `minecraftVersion` 是一个 C 风格的字符串，内容为当前 *Minecraft* 的版本名称。
- `nmodApiVersion` 是一个 C 风格的字符串，内容为 NModAPI 的版本名称。
- `pathOflibminecraftpeso` 是一个 C 风格字符串，内容为 `libminecraftpe.so` 的路径。 你可以使用 `dlopen` 装载它：`dlopen(pathOflibminecraftpeso, RTLD_LAZY)` 。

### 游戏启动 （OnActivityCreate）

```cpp
NMod_OnActivityCreate(JNIEnv* env, jobject thiz, jobject savedInstanceState)
```
- `env` 是一个指向 `JNIEnv` 的指针。
- `thiz` 是一个 `jobject` ，Java 类型签名为 `[Lcom/mojang/minecraftpe/MainActivity;]` 。
- `savedInstanceState` 是一个 `jobject` ，Java 类型签名为 `[Landroid/os/Bundle;]` 。
  
### 游戏退出（OnActivityFinish）

```cpp
NMod_OnActivityFinish(JNIEnv* env, jobject thiz)
```

- `env` 是一个指向 `JNIEnv` 的指针。
- `thiz` 是一个 `jobject` ，Java 类型签名为 `[Lcom/mojang/minecraftpe/MainActivity;]` 。
  

## 修改内置方法
经过以上的步骤后，我们该如何修改 *Minecraft* 中的内置方法？

*Substrate* 框架提供了一个修改的方法—— `MSHookFunction` ，该方法可以用我们自己定义的方法替换 `libminecraftpe.so` 中定义的默认方法，也提供了一个调用默认方法的途径。

调用 `MSHookFunction` 方法需要三个参数：
```cpp
MSHookFunction( 
    (void*)& DefaultMethod,
    (void*)& ReplacementMethod,
    (void**)& MethodPointerOfDefaultMethod
);
```
例如，如果我们想要替换 `libminecraftpe.so` 中的 `Explosion::explode()` 方法，我们可以这样写：

```cpp
// 定义默认方法（原方法）
class Explosion
{
    public:
        void explode();
}

// 定义一个函数指针
void (*explode_default)(Explosion*);
void explode_replacement(Explosion* self)
{
    // Do Something

    // 如果不想忽略这次爆炸，请调用函数指针：
    explode_default(self);
}

// 在 NMod_OnLoad 中注册 MSHookFunction 方法
extern "C" void NMod_OnLoad(JavaVM*,JNIEnv*,const char*,const char*,const char*)
{
    MSHookFunction((void*)& Explosion::explode,
                (void*)& explode_replacement,
                (void**)& explode_default);
}
```

## NMod 配置

NModAPI 通过读取 `nmod_manifest.json` 来获取 NMod 配置。你也可以在配置中编辑 `assets` 下的 Json 与文本。

```json
{
  // 提示：你不用定义该配置文件中的所有内容。
  
  // NMod 名称
  "name" : "My NMod",
  
  // NMod 的包名，必须与 AndroidManifest.xml 中的包名一致。
  "package_name" : "my.nmod.package.name",
  
  // NMod 的作者，在此写下你的名字或组织名。
  "author" : "Me or My Company",
  
  // NMod 原生库
  "native_libs_info" :
  [
    {
      // 原生库名，必须与你的库名一致。
      "name" : "libmynmodlib.so",

      // 该 NMod 是否使用 NModAPI
      // 如果你使用了事件监听器（NMod_OnLoad, NMod_OnActivityCreate,
      //   NMod_OnActivityFinish），use_api 必须为 true 。
      "use_api" : "true"
    }
    
    // 你还可以在此加载更多库
    //,
    //{
    //  "name" : "libmynmodlib2.so",
    //  "use_api" : false
    //}
  ],
  
  // NMod 版本号
  // 默认值为 -1 。
  "version_code" : 1,
  
  // NMod 版本名
  "version_name" : "1.0",

  // Minecraft 版本名
  "minecraft_version_name" : "1.11.0.1",
  
  // NMod 描述
  "description" : "My NMod Description",
  
  // 更新内容
  "change_log" : "My NMod Change Log",
  
  
  // NMod Banner
  // NMod Banner 是在主界面展示的一张图像。
  
  // 图像下方的文本视图，默认值为 NMod 名称（name）。
  "banner_title" : "My NMod is the Best!",
  
  // 在 assets 文件夹内的图像路径
  // 图像大小必须为 1024（长） * 500（宽）。
  // 在当前设置下，NMod 将会寻找 assets/my_banner.png 。
  "banner_image_path" : "my_banner.png"
  
  // 材质文本编辑 API
  // text_edit 允许你编辑 Minecraft PE 中材质的文本文件。
  
  /*
  ,
  "text_edit" :
  [
    {
      // assets 中要修改的的文本文件路径
      // 要替换的文件与默认文件（源文件）必须同时在 NMod 与 Minecraft PE 的 assets 中定义。

      // 在当前设置下，NModAPI 将会读取 assets/resource_packs/vanilla/texts/en_US.lang 。
      "path" : "resource_packs/vanilla/texts/en_US.lang",
      
      // 有三种编辑模式：append（追加）, prepend（前置）, replace（替换），默认值为 replace 。
      // 如果你定义了一种不存在的模式，编辑将不起作用。
      // append：在源文件后追加文本。
      // prepend：在源文件前追加文本。
      // replace: 用 NMod assets 中定义的新文本替换源文本。

      // 在当前设置下，NModAPI 将会在默认文本（源文本）后追加要替换的文本。
      "mode" : "append"
    },
    {
      "path" : "resource_packs/vanilla/texts/zh_CN.lang",
      "mode" : "append"
    }
  ]*/
  
  // 材质 Json 编辑 API
  // json_edit 允许你编辑 Minecraft PE 中材质的 Json 文件。
  
  /*
  ,
  "json_edit" :
  [
    {
      // assets 中要修改的 Json 文件路径。
      // 要替换的文件与默认文件（源文件）必须同时在 NMod 与 Minecraft PE 的 assets 中定义。
      // 在当前设置下，NModAPI 将会读取 resource_packs/vanilla/textures/item_texture.json
      "path" : "resource_packs/vanilla/textures/item_texture.json",
      
      // 有两种编辑模式：merge（合并），replace（替换），默认值为替换。
      // 如果你定义了一种不存在的模式，编辑将不起作用。
      // merge：合并两个 Json 文件。
      // replace：用 NMod assets 中定义的新 Json 替换源 Json。
      // 在当前情况下，NModAPI 将会合并两个 Json 为一个。
      "mode" : "merge"
    }
    
    // 你还可以在此编辑更多的 Json
    //,
    //{
    //  "path" : "File Path",
    //  "mode" : "replace"
    //}
  ]
  */
}
```

## 打包你的 NMod 
-  打包成 APK
   
   如果你的 NMod 被打包成一个 APK 文件，它可以被 Android 软件包管理器安装且能被 NModAPI 读取。
   
   在这种情况下，`nmod_manifest.json` 应该放入 `assets` 文件夹内。原生库应该打包进 `lib/[对应的 CPU 架构]` 内。

   提示：
      - NModAPI 仅读取两种架构：`armeabi-v7a` 以及 `x86` 。
      - 安装新版本的 NMod 时，使用 Android 软件包管理器的 NMod 可以自动更新。所以这种方式大部分用于开发 NMod 。
  
   注意：
      - 在 `nmod_manifest.json` 中定义的 `package_name` 必须与 `AndroidManifest.xml` 中定义的 `package` 属性一致。
  
- 打包成文件
  
  不喜欢安装 APK ？你还可以将你的 NMod 打包成一个文件！它的扩展名可以为 `*.apk`, `*.zip`, `*.nmod`, `*.mcnmod` 。

  在这种情况下，`nmod_manifest.json` 可以放入 `assets` 文件夹或 NMod 的根目录。

  提示：
  - 打包成文件的 NMod 不能自动更新，所以这种方式大部分用于发布 NMod 。

  注意：
  - 不要在不同路径中同时放置两个 `nmod_manifest.json` ！
  
## 仍然不理解？
- 查阅我们的示例！
<https://github.com/TimScriptov/NMOD-Examples>
- 查阅 NModAPI 的源代码！
<https://github.com/TimScriptov/ModdedPE>
