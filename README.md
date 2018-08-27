# NModAPI Document

## What is NMod?<br>
NMod's full name is Native-NMod.Native-Mod modifies Minecraft with native code(c/c++),so it is called NMod.As we all know,Minecraft(Pocket Edition) is mainly made with c++.To get better modification effects than modpe scripts,why don't you learn to make NMods?<br>
But,NMods isn't easy to make.C++ Programming Language is much more difficult that JavaScript,and native modification methods is hard to understand.The following will tell you how to develop NMods.<br>

## Preparation<br>
1.C++ Programming Language skills<br>
2.An android native code compiler(Android Studio,Eclipse,AIDE,ect.)<br>
3.Some skills to build shared native libraries(*.so)<br>
4.Json Syntax knowledges.<br>

## Linked Libraries<br>
Native-Mods links libsubstrate.so and libminecraftpe.so to perform modifications.Make sure you have linked module substrate and minecraftpe.<br>
(You can view our examples to know how to link these libraries.)<br>

## Game Listeners<br>
NModAPI provides some listeners to tell each NMod when the nmod is loaded,the game is started or finished.You only need to define these methods,then NModAPI will invoke them.<br>
- OnLoad Listener:<br>
NMod_OnLoad(JavaVM* javaVM,JNIEnv* jniEnv,const char* minecraftVersion,const char* nmodApiVersion,const char* pathOflibminecraftpeso)<br>
javaVM is a pointer to JavaVM.<br>
jniEnv is a pointer to JNIEnv.<br>
minecraftVersion is a c-style string,values installed minecraft version name.<br>
nmodApiVersion is a c-style string,values NModAPI version name.<br>
pathOflibminecraftpeso is a c-style string,values libminecraftpe.so's path.You can use dlopen with it: dlopen(pathOflibminecraftpeso,RTLD_LAZY);<br>
- OnActicityCreate Listener:<br>
NMod_OnActivityCreate(JNIEnv* env,jobject thiz,jobject savedInstanceState)<br>
env is a pointer to JNIEnv<br>
thiz is a jobject.[Lcom/mojang/minecraftpe/MainActivity;]<br>
savedInstanceState is a jobject.[Landroid/os/Bundle;]<br>
- OnActivityFinish Listener:<br>
NMod_OnActivityFinish(JNIEnv* env,jobject thiz)<br>
env a ppinter to JNIEnv<br>
thiz is a jobject.[Lcom/mojang/minecraftpe/MainActivity;]<br>
<br>
Haven't understand?View our examples!<br>

## Modification Methods<br>
After the above steps,how can we modify minecraft?<br>
Substrate Framework provides modification method:MSHookFunction.MSHookFunction can replace default methods defines in libminecraftpe.so with our own methods,and offers a way to invoke default methods.<br>
MSHookFunction requires three arguments:<br>
MSHookFunction( (void*)& DefaultMethod,<br>
                (void*)& ReplacementMethod,<br>
                (void**)& MethodPointerOfDefaultMethod);<br>
For example,if we want to replace Explosion::explode() in libminecraftpe.so,we can write:

```cpp
//Define Default Method
class Explosion
{
    public:
        void explode();
}
//Define Method Pointer
void (*explode_default)(Explosion*);
void explode_replacement(Explosion* self)
{
    //Do Something
    //If you want to explode properly instead of no explosion,invoke the method pointer.
    explode_default(self);
}
//Register MSHookFunction in NMod_OnLoad
extern "C" void NMod_OnLoad(JavaVM*,JNIEnv*,const char*,const char*,const char*)
{
    MSHookFunction( (void*)& Explosion::explode,
                (void*)& explode_replacement,
                (void**)& explode_default);
}
```
## NMod Manifest<br>
NModAPI read nmod information by reading nmod_manifest.json.<br>
If you want to use nmod json editor or nmod text editor,you can also define edit info in it!<br>
```json
{
  //Tips : You needn't to define all elements in this json file.
  
  //NMod name.
  "name" : "My NMod",
  
  //NMod package name.Must equal to package name defines in AndroidManifest.xml
  "package_name" : "my.nmod.package.name",
  
  //Author of your NMod.Write your name or your organization name here.
  "author" : "Me or My Company",
  
  //NMod native libraries.
  "native_libs_info" :
  [
    {
      //native library name:
      //must match your native library name!
      "name" : "libmynmodlib.so",
      //whether the nmod uses nmodapi:
      //use_api must be true if you use game listeners(NMod_OnLoad,NMod_OnActivityCreate,NMod_OnActivityFinish).
      "use_api" : "true"
    }
    
    //You can even load more libraries...
    //,
    //{
    //  "name" : "libmynmodlib2.so",
    //  "use_api" : false
    //}
  ],
  
  //Version Code:
  //Default value is -1.
  "version_code" : 1,
  
  //Version Name:
  "version_name" : "1.0",
  
  //Nmod description:
  "description" : "My NMod Description",
  
  //What's new in this version?Write them in change_log.
  "change_log" : "My NMod Change Log",
  
  
  //NMod Banner
  //NMod Banner is a image view showed in main page.
  
  //Banner title is a text view lays under the image.It's default value is NMod name.
  "banner_title" : "My NMod is the Best!",
  
  //The file path of banner image in assets.
  //The image must be the size of 1024(width)*500(height)!
  //NModAPI will find assets/my_banner.png in this case.
  "banner_image_path" : "my_banner.png"
  
  //Assest Text Edit API
  //Text edit can edit text files in minecraftpe assets.
  
  /*
  ,
  "text_edit" :
  [
    {
      //path is the text file path in assets.
      //The replacement file and the default file must defines in nmod assets and minecraft assets at the same time.
      //In this case,NModAPI will read assets/resource_packs/vanilla/texts/en_US.lang
      "path" : "resource_packs/vanilla/texts/en_US.lang",
      
      //There are three kinds of text edit mode: append,prepend,replace.
      //Default value is replace.
      //If you defines a different mode,THE TEXT EDIT WILL NOT WORK!
      //MODE: append: append text after the source text.
      //MODE: prepend: prepend text before the source text.
      //MODE: replace: replace the source text with the new one(text defines in nmod assets).
      //In this case,NModAPI will append the replacement texts after the default texts.
      "mode" : "append"
    },
    {
      "path" : "resource_packs/vanilla/texts/zh_CN.lang",
      "mode" : "append"
    }
  ]*/
  
  //Assest Json Edit API
  //Json edit can edit json files in minecraftpe assets.
  
  /*
  ,
  "json_edit" :
  [
    {
      //path is json file path in assets.
      //The replacement file and the default file must defines in nmod assets and minecraft assets at the same time.
      //In this case,NModAPI will read resource_packs/vanilla/textures/item_texture.json
      "path" : "resource_packs/vanilla/textures/item_texture.json",
      
      //There are two kinds of json edit mode: merge,replace.
      //Default value is replace.
      //If you defines a different mode,THE TEXT EDIT WILL NOT WORK!
      //MODE: merge: merge the two json files.
      //MODE: replace: replace the source json with the new one(json defines in nmod assets).
      //In this case,NModAPI will merge the two json files into one.
      "mode" : "merge"
    }
    
    //You can even edit more json...
    //,
    //{
    //  "path" : "File Path",
    //  "mode" : "replace"
    //}
  ]
  */
}
```
## Pack Your NMod!
- Way 1: Packing into apk.
If you pack your nmod into a apk file,it can be installed by android package manager and read by NModAPI.<br>
In this case,nmod_manifest.json should be put into assets.(assets/nmod_manifest.json).Native Libraries should be packed into lib/CPU_ARCH/.<br>
Tips: NModAPI only read two kind of CPU_ARCH : armeabi-v7a and x86.<br>
Tips: NMod installed by Android Package Manager can auto update if you install a new version nmod.So packing into apk is mostly used for developing NMods.<br>
Warning: package_name defines in nmod_manifest.json must equal to package defines in AndroidManifest.xml!<br>
- Way 2: Packing into file.
Don't like installing apk?You can even pack your NMod into a file!<br>
This file can be (*.apk,*.zip,*.nmod,*.mcnmod).<br>
In this case,nmod_manifest.json can be put into assets dir or root dir(assets/nmod_manifest.json or nmod_manifest.json).<br>
Warning: DO NOT put two nmod_manifest.json in different directories!<br>
Tips: NMod packed into file cannot auto update.So it is mostly used for publishing NMods.
## Haven't Understand?
- View our NMod Examples!<br>
<https://github.com/ModelPart/NMOD-Examples>
- View the source code of NModAPI!<br>
<https://github.com/ModelPart/ModdedPE>
