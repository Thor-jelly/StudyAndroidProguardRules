# StudyAndroidProguardRules
学习android代码混淆
#代码混淆
> 如有错误可以QQ邮箱联系，745661590@qq.com  
> github不支持脚注

# 代码混淆概念[^1]
代码混淆(Obfuscated code)亦称花指令，是将计算机程序的代码，转换成一种功能上等价，但是难于阅读和理解的形式的行为。代码混淆可以用于程序源代码，也可以用于程序编译而成的中间代码。执行代码混淆的程序被称作代码混淆器。目前已经存在许多种功能各异的代码混淆器。

# 混淆配置
我们一般直接在app的build.gradle文件中设置minifyEnabled true即可;  
我们一般也加上shrinkResources true，打开资源压缩。用shrinkResources true开启资源压缩后，所有未被使用的资源默认被移除。

```
android {
    buildTypes {
        release {
            minifyEnabled true//是否启动混淆true:打开;false:关闭
            shrinkResources true//打开资源压缩。
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

# 设置白名单
白名单也就是proguard-rules.pro中我们自定义哪些不添加混淆。

## 关键字[^2]
|关键字|描述|
|:-----:|:-----:|
|dontwarn|  dontwarn是一个和keep可以说是形影不离,尤其是处理引入的library时.|
|keep|  保留类和类中的成员，防止被混淆或移除|
|keepnames| 保留类和类中的成员，防止被混淆，成员没有被引用会被移除|
|keepclassmembers|  只保留类中的成员，防止被混淆或移除|
|keepclassmembernames|  只保留类中的成员，防止被混淆，成员没有引用会被移除|
|keepclasseswithmembers|    保留类和类中的成员，防止被混淆或移除，保留指明的成员|
|keepclasseswithmembernames|    保留类和类中的成员，防止被混淆，保留指明的成员，成员没有引用会被移除|

```
[保持命令] [类] {
    [成员] 
}
```

## 通配符
|通配符|描述|
|:-----:|:-----:|
|*| 匹配任意长度字符，不包含包名分隔符(.)|
|**|    匹配任意长度字符，包含包名分隔符(.)|
|***|   匹配任意参数类型|
|$| 内部类|
|\<init>|    匹配类中所有的构造函数|
|\<fields>|  匹配类中的所有字段|
|\<method>|  匹配类中所有的方法|
|…| 匹配任意长度的任意类型参数。比如void test(…)就能匹配任意 void test(String a) 或者是 void test(int a, String b) 这些方法。|
|extends|   即可以指定类的基类|
|implement|     匹配实现了某接口的类|

## 规则使用
- 不混淆某个类  

    ```
    -keep public class com.dongdongwu.thor.entry.Test { *; }
    ```
- 不混淆某个包所有的类

    ```
    -keep class com.dongdongwu.thor.entry.** { *; }
    ```
- 不混淆某个类的子类

    ```
    -keep public class * extends com.dongdongwu.thor.entry.Test { *; }
    ```
- 不混淆所有类名中包含了“model”的类及其成员

    ```
    -keep public class **.*model*.** { *; }
    ```
- 不混淆某个接口的实现

    ```
    -keep class * implements com.dongdongwu.thor.entry.Test { *; }
    ```
- 不混淆某个类的构造方法

    ```
    -keepclassmembers class com.dongdongwu.thor.entry.Test {
        public <init>(); 
    }
    ```
- 不混淆某个类的特定的方法

    ```
    -keepclassmembers class com.dongdongwu.thor.entry.Test {
        public void test(java.lang.String); 
    }
    ```
- 不混淆某个类的内部类

    ```
    -keep class com.dongdongwu.thor.entry.Test$* {
        *; 
    }
    ```

## 我总结的

> 最新的proguard-android.txt中添加了很多下面已经有的，再添加的时候看看是不是已经有了

```

#######################-----app中-----############################
#如果使用了Gson之类的工具要使被它解析的JavaBean类即实体类不被混淆

#######################-----第三方-----############################

#######################-----WebView(项目中没有可以忽略)-----############################
#webView需要进行特殊处理
-keepclassmembers class fqcn.of.javascript.interface.for.Webview {
   public *;
}
-keepclassmembers class * extends android.webkit.WebViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.WebViewClient {
    public void *(android.webkit.WebView, jav.lang.String);
}
#在app中与HTML5的JavaScript的交互进行特殊处理
#我们需要确保这些js要调用的原生方法不能够被混淆，于是我们需要做如下处理：
-keepclassmembers class com.ljd.example.JSInterface {
    <methods>;
}


#######################-----其他-----############################
# 删除代码中Log相关的代码
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int i(...);
    public static int w(...);
    public static int d(...);
    public static int e(...);
}

# 保持测试相关的代码
-dontnote junit.framework.**
-dontnote junit.runner.**
-dontwarn android.test.**
-dontwarn android.support.test.**
-dontwarn org.junit.**

#######################-----基本-----############################
#指定代码的压缩级别 0 - 7(指定代码进行迭代优化的次数，在Android里面默认是5，这条指令也只有在可以优化时起作用。)
-optimizationpasses 5

#混淆时不会产生形形色色的类名(混淆时不使用大小写混合类名,混淆后类名都为小写)
-dontusemixedcaseclassnames

#指定不去忽略非公共的库的类
#默认跳过，有些情况下编写的代码与类库中的类在同一个包下，并且持有包中内容的引用，此时就需要加入此条声明
-dontskipnonpubliclibraryclasses
#指定不去忽略非公共的库的类的成员
-dontskipnonpubliclibraryclassmembers

#Optimization is turned off by default. Dex does not like code run
#hrough the ProGuard optimize and preverify steps (and performs some
#of these optimizations on its own).
#不进行优化，建议使用此选项，不优化输入的类文件（原因见上边的原英文注释）
-dontoptimize
#不做预检验，preverify是proguard的四个步骤之一
#Android不需要preverify，去掉这一步可以加快混淆速度
-dontpreverify

#屏蔽警告
-ignorewarnings

#指定混淆是采用的算法，后面的参数是一个过滤器
#这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

#避免混淆Annotation、内部类、泛型、匿名类
-keepattributes *Annotation*,InnerClasses,Signature,EnclosingMethod

#表示不混淆声明的两个类，这两个类我们基本也用不上，是接入Google原生的一些服务时使用的
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

#抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable
#重命名抛出异常时的文件名称,点击鼠标可以到达错误文件位置
-renamesourcefileattribute SourceFile

#混淆时是否记录日志
-verbose

#不混淆R文件中的所有静态字段，以保证正确找到每个资源的id
-keepclassmembers class **.R$* {
    public static <fields>;
}
 
#保留四大组件，自定义的Application等这些类不被混淆
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View

#处理support包
-keep class android.support.** {*;}
-dontnote android.support.**
-dontwarn android.support.**

#保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

#保留本地native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

#保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

#保持 Serializable 不被混淆并且enum 类也不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

#保留Parcelable序列化类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

#see http://proguard.sourceforge.net/manual/examples.html#beans
#不混淆View中的setXxx()和getXxx()方法，以保证属性动画正常工作
-keep public class * extends android.view.View{
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

#不混淆Activity中参数是View的方法，例如，一个控件通过android:onClick="clickMethodName"绑定点击事件，混淆后会导致点击事件失效
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

#保留Keep注解的类名和方法
-keep class android.support.annotation.Keep
-keep @android.support.annotation.Keep class * {*;}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}

#第三方jar包不被混淆
-keep class com.github.** {*;}
```

# 参考文档
- [Android 混淆那些事儿](https://zhuanlan.zhihu.com/p/27582991)
- [5分钟搞定android混淆](https://www.jianshu.com/p/f3455ecaa56e)
- [写给Android开发者的混淆使用手册](https://www.jianshu.com/p/158aa484da13)
- [Android混淆](https://www.jianshu.com/p/b5b2a5dfaaf4)
- [Android 混淆解析](https://www.jianshu.com/p/84114b7feb38)
- [Android混淆总结篇(一)](http://blog.csdn.net/yk377657321/article/details/60501880)

[^1]: [摘抄自百度百科-代码混淆](https://baike.baidu.com/item/%E4%BB%A3%E7%A0%81%E6%B7%B7%E6%B7%86/1892288?fr=aladdin)
[^2]: 摘抄自[Android混淆](https://www.jianshu.com/p/b5b2a5dfaaf4)的关键字描述
