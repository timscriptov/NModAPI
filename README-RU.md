# Документация NModAPI

## Что такое NMod?<br>
Полное имя NMod - Native-NMod. Native-Mod модифицирует Minecraft с помощью собственного кода (C/C++), поэтому он называется NMod. Как мы все знаем, Minecraft Pocket Edition в основном написан на C++. Чтобы получить лучшие эффекты модификации, чем modpe scripts, почему бы вам не научиться разрабатывать NMOD?<br>
Но NMods нелегко сделать. Язык программирования C++ очень сложный. Ниже рассказывается, как разработать NMods.<br>

## Подготовка<br>
1. Навыки программирования на C++<br>
2. Компилятор кода для Android (Android Studio, Eclipse, AIDE и др.)<br>
3. Некоторые навыки создания библиотек (*.so)<br>
4. Знание синтаксиса Json.<br>

## Объединение библиотек<br>
Native-Mods ссылается на libsubstrate.so и libminecraftpe.so для выполнения модификаций. Убедитесь, что вы объеденили модуль substrate и файл minecraftpe.<br>
(Вы можете просмотреть наши примеры, чтобы узнать, как связать эти библиотеки.)<br>

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
Не понимаете? Посмотрите наши примеры!<br>

## Методы модификации<br>
После вышеуказанных шагов, как мы можем изменить minecraft?<br>
Substrate Framework предоставляет метод модификации: MSHookFunction.MSHookFunction может заменить методы по умолчанию, заданные в libminecraftpe.so нашими собственными методами, и предлагает способ вызова методов по умолчанию.<br>
MSHookFunction требует три аргумента:<br>
MSHookFunction( (void*)& DefaultMethod,<br>
                (void*)& ReplacementMethod,<br>
                (void**)& MethodPointerOfDefaultMethod);<br>
Например, если мы хотим заменить взрыв :: explode() в libminectpepe.so, мы можем написать:

```cpp
//Определить метод по умолчанию
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
//Регистрация MSHookFunction в NMod_OnLoad
extern "C" void NMod_OnLoad(JavaVM*,JNIEnv*,const char*,const char*,const char*)
{
    MSHookFunction( (void*)& Explosion::explode,
                (void*)& explode_replacement,
                (void**)& explode_default);
}
```
## NMod Manifest<br>
NModAPI читает информацию nmod, читая nmod_manifest.json.<br>
Если вы хотите использовать редактор nmod json или текстовый редактор nmod, вы также можете определить информацию редактирования в нём!<br>
```json
{
  //Советы. Вам не нужно определять все элементы в этом json-файле.
  
  //Имя NMod.
  "name" : "My NMod",
  
  //Имя пакета NMod. Имя пакета должно соответствовать пакету в AndroidManifest.xml!
  "package_name" : "my.nmod.package.name",
  
  //Автор NMod. Напишите здесь свое имя или название вашей организации.
  "author" : "Me or My Company",
  
  //NMod native libraries.
  "native_libs_info" :
  [
    {
      //имя native library:
      //должен соответствовать имени вашей native library!
      "name" : "libmynmodlib.so",
      //whether the nmod uses nmodapi:
      //use_api must be true if you use game listeners(NMod_OnLoad,NMod_OnActivityCreate,NMod_OnActivityFinish).
      "use_api" : "true"
    }
    
    //Вы даже можете загружать больше библиотек ...
    //,
    //{
    //  "name" : "libmynmodlib2.so",
    //  "use_api" : false
    //}
  ],
  
  //Код версии:
  //Default value is -1.
  "version_code" : 1,
  
  //Версия:
  "version_name" : "1.0",
  
  //Описание Nmod:
  "description" : "My NMod Description",
  
  //Что нового в этой версии? Напишите изменения в change_log.
  "change_log" : "My NMod Change Log",
  
  
  //NMod Banner
  // Mod Banner - это изображение на главной странице.
  
  //Заголовок баннера - это текстовое представление, лежащее под изображением. По умолчанию это имя NMOD.
  "banner_title" : "My NMod is the Best!",
  
  //Путь к файлу баннера в assets.
  //Изображение должно быть размером 1024 (ширина) * 500 (высота)!
  //NModAPI will find assets/my_banner.png in this case.
  "banner_image_path" : "my_banner.png"
  
  //Assest Text Edit API
  //Текстовый редактор может редактировать текстовые файлы в файлах minecraft pe.
  
  /*
  ,
  "text_edit" :
  [
    {
      //path - путь текстового файла в assets.
      //Файл замены и файл по умолчанию должны определяться в assets nmod и в assets minecraft одновременно.
      //В этом случае NModAPI будет читать assets/resource_packs/vanilla/texts/en_US.lang
      "path" : "resource_packs/vanilla/texts/en_US.lang",
      
      //Существует три вида режима редактирования текста: объединение, prepend, замена.
      //Значение по умолчанию - замена.
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
  //Редактор Json может редактировать json-файлы в minecraft assets.
  
  /*
  ,
  "json_edit" :
  [
    {
      //path - путь к файлу json в assets.
      //Файл замены и файл по умолчанию должны определяться в assets nmod и в assets minecraft одновременно.
      //В этом случае NModAPI будет читать resource_packs/vanilla/textures/item_texture.json
      "path" : "resource_packs/vanilla/textures/item_texture.json",
      
      //Существует два режима редактирования json: слияние, замена.
      //Значение по умолчанию - замена.
      //If you defines a different mode,THE TEXT EDIT WILL NOT WORK!
      //MODE: merge: объединяет два json-файла.
      //MODE: replace: замените источник json на новый (json определяет в ресурсах nmod).
      //В этом случае NModAPI объединит два json-файла в один.
      "mode" : "merge"
    }
    
    //Вы даже можете отредактировать json...
    //,
    //{
    //  "path" : "File Path",
    //  "mode" : "replace"
    //}
  ]
  */
}
```
## Упакуйте свой NMod!
- Way 1: Упаковка в apk.
Если вы упакуете nmod в файл apk, его можно установить менеджером пакетов Android и прочитать NModAPI.<br>
В этом случае nmod_manifest.json следует поместить в активы. (assets/nmod_manifest.json). Библиотеки должны быть упакованы в lib/CPU_ARCH/.<br>
Советы: NModAPI читает только два вида CPU_ARCH: armeabi-v7a и x86.<br>
Советы: NMod, установленный диспетчером Android Package, может автоматически обновляться, если вы устанавливаете новую версию nmod. Так что упаковка в apk в основном используется для разработки NMods.<br>
Предупреждение: package_name определяется в nmod_manifest.json, которое должно соответствовать пакету в AndroidManifest.xml!<br>
- Way 2: Упаковка в файл.
Не нравится установка apk? Вы можете упаковать свой NMOD в файл!<br>
Этот файл может быть (*.apk, *.zip, *.nmod, *.mcnmod).<br>
В этом случае nmod_manifest.json можно поместить в активы dir или root dir (assets/nmod_manifest.json или nmod_manifest.json).<br>
Предупреждение: НЕ помещайте два nmod_manifest.json в разные каталоги! <br>
Советы: NMod, упакованный в файл, не может автоматически обновляться. Поэтому он в основном используется для публикации NMOD.
## Не поняли?
- Просмотрите наши примеры NMod!<br>
<https://github.com/TimScriptov/NMOD-Examples>
- Просмотр исходного кода NModAPI!<br>
<https://github.com/TimScriptov/ModdedPE>
