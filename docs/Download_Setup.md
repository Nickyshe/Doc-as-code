## **Download and SetUp**
* [Android SDK Requirements](#android-sdk-requirements)
* [Download](#download)
    * [Gradle](#gradle)
    * [Maven](#maven)
* [Setup](#setup)
    * [Permissions](#permissions)
        * [Internet](#internet)
        * [Connectivity Monitoring](#connectivity-monitoring)
        * [Local Storage](#local-storage)
    * [Proguard](#proguard)
    * [Java 8](#java-8)
    * [Configuring Glide / Annotation Processors](#configuring-glide--annotation-processors)
        * [Java](#java)
        * [Kotlin - KAPT](#kotlin---kapt)
        * [Kotlin - KSP](#kotlin---kapt)


### **Android SDK Requirements**

**Minimum SDK Version** - Glide requires a minimum SDK version of **14** (Ice Cream Sandwich) or higher.

**Compile SDK Version** - Glide must be compiled against SDK version **27** (Oreo MR1) or higher.


### **Download**

Glide’s public releases are accessible via Maven and Gradle.


#### **Gradle**

If you use Gradle you can add a dependency on Glide using either Maven Central or JCenter. You will also need to include a dependency on the support library.


```
repositories {
  google()
  mavenCentral()
}

dependencies {
    implementation 'com.github.bumptech.glide:glide:4.14.2'
    // Skip this if you don't want to use integration libraries or configure Glide.
    annotationProcessor 'com.github.bumptech.glide:compiler:4.14.2'
}
```


**Note:** Avoid using `@aar` in your dependencies whenever possible. If you must do so, add `transitive = true` to ensure that all necessary classes are included in your APK:


```
dependencies {
    implementation ("com.github.bumptech.glide:glide:4.14.2@aar") {
        transitive = true
    }
}
```


`@aar` is Gradle’s [“Artifact only”](https://docs.gradle.org/current/userguide/dependency_management.html#ssub:artifact_dependencies) notation that excludes dependencies by default.

Excluding Glide’s dependencies by using `@aar` without `transitive = true `will result in runtime exceptions like:


```
java.lang.NoClassDefFoundError: com.bumptech.glide.load.resource.gif.GifBitmapProvider
    at com.bumptech.glide.load.resource.gif.ByteBufferGifDecoder.<init>(ByteBufferGifDecoder.java:68)
    at com.bumptech.glide.load.resource.gif.ByteBufferGifDecoder.<init>(ByteBufferGifDecoder.java:54)
    at com.bumptech.glide.Glide.<init>(Glide.java:327)
    at com.bumptech.glide.GlideBuilder.build(GlideBuilder.java:445)
    at com.bumptech.glide.Glide.initializeGlide(Glide.java:257)
    at com.bumptech.glide.Glide.initializeGlide(Glide.java:212)
    at com.bumptech.glide.Glide.checkAndInitializeGlide(Glide.java:176)
    at com.bumptech.glide.Glide.get(Glide.java:160)
    at com.bumptech.glide.Glide.getRetriever(Glide.java:612)
    at com.bumptech.glide.Glide.with(Glide.java:684)
```



#### **Maven**

If you use Maven you can add a dependency on Glide as well. Again, you will also need to include a dependency on the support library.


```
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>glide</artifactId>
  <version>4.14.2</version>
  <type>aar</type>
</dependency>
<dependency>
  <groupId>com.google.android</groupId>
  <artifactId>support-v4</artifactId>
  <version>r7</version>
</dependency>
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>compiler</artifactId>
  <version>4.14.2</version>
  <optional>true</optional>
</dependency>
```



### **Setup**

Depending on your build configuration you may also need to do some additional setup.


#### **Permissions**

Glide does not require any permissions out of the box assuming all of the data you’re accessing is stored in your application. That said, most applications either load images on the device (in DCIM, Pictures or elsewhere on the SD card) or load images from the internet. As a result, you’ll want to include one or more of the permissions listed below, depending on your use cases.


##### **Internet**

However if you’re planning on loading images from urls or over a network connection, you should add the `INTERNET` and `ACCESS_NETWORK_STATE` permissions to your `AndroidManifest.xml`:


```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="your.package.name"

    <uses-permission android:name="android.permission.INTERNET"/>
    <!--
    Allows Glide to monitor connectivity status and restart failed requests if users go from a
    a disconnected to a connected network state.
    -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <application>
      ...
    </application>
</manifest>
```


`ACCESS_NETWORK_STATE` isn’t technically required to allow Glide to load urls, but it helps Glide handle flaky network connections and airplane mode. See the Connectivity Monitoring section below for more details


##### **Connectivity Monitoring**

If you’re loading images from urls, Glide can automatically help you deal with flaky network connections by monitoring users’ connectivity status and restarting failed requests when users are reconnected. If Glide detects that your application has the `ACCESS_NETWORK_STATE`, Glide will automatically monitor connectivity status and no further changes are needed.

You can verify that Glide is monitoring network status by checking the `ConnectivityMonitor` log tag:


```
adb shell setprop log.tag.ConnectivityMonitor DEBUG
```


After doing so, if you’ve successfully added the `ACCESS_NETWORK_STATE` permission, you will see logs in logcat like:


```
11-18 18:51:23.673 D/ConnectivityMonitor(16236): ACCESS_NETWORK_STATE permission granted, registering connectivity monitor
11-18 18:48:55.135 V/ConnectivityMonitor(15773): connectivity changed: false
11-18 18:49:00.701 V/ConnectivityMonitor(15773): connectivity changed: true
```


If the permission is missing, you’ll see an error instead:


```
11-18 18:51:23.673 D/ConnectivityMonitor(16236): ACCESS_NETWORK_STATE permission missing, cannot register connectivity monitor
```



##### **Local Storage**

To load images from local folders like DCIM or Pictures, you’ll need to add the `READ_EXTERNAL_STORAGE` permission:


```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="your.package.name"

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

    <application>
      ...
    </application>
</manifest>
```


To use <code>[ExternalPreferredCacheDiskCacheFactory](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/cache/ExternalPreferredCacheDiskCacheFactory.html)</code> to store Glide’s cache on the public sdcard, you’ll need to use the <code>WRITE_EXTERNAL_STORAGE</code> permission instead:


```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="your.package.name"

    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application>
      ...
    </application>
</manifest>
```



#### **Proguard**

Proguard configurations are included with the Glide library, but if you need to customize, see [Glide’s proguard file](https://github.com/bumptech/glide/blob/master/library/proguard-rules.txt)


#### **Java 8**

Starting with Android Studio 3.0 and version 3.0 of the Android Gradle plugin, you can compile your project and Glide with Java 8. For details, see the [Use Java 8 Language Features](https://developer.android.com/studio/write/java8-support.html) on the Android Developers website.

Glide requires Java 11 to compile, but produces Java 7 compatible source (except for tests and samples).


#### **Configuring Glide / Annotation Processors**

To [configure](https://bumptech.github.io/glide/doc/configuration.html) Glide, you’ll need to include one of Glide’s annotation processing libraries.


##### **Java**

If you’re using Java, including Glide’s annotation processor requires dependencies on Glide’s annotations and the annotation processor:


```
compile 'com.github.bumptech.glide:annotations:4.14.2'
annotationProcessor 'com.github.bumptech.glide:compiler:4.14.2'
```



##### **Kotlin - KAPT**

If you use Glide’s annotations on classes implemented in Kotlin, you can include a `kapt` dependency on Glide’s annotation processor instead of a `annotationProcessor` dependency:


```
dependencies {
  kapt 'com.github.bumptech.glide:compiler:4.14.2'
}
```


Note that you must also include the `kotlin-kapt` plugin in your `build.gradle` file:


```
apply plugin: 'kotlin-kapt'
```


Keep in mind that if you have any other annotation processors, all of them must be converted from `annotationProcessor` to `kapt`:


```
dependencies {
  kapt "android.arch.lifecycle:compiler:1.0.0"
  kapt 'com.github.bumptech.glide:compiler:4.14.2'
}
```


For more details on `kapt`, see the [official documentation](https://kotlinlang.org/docs/reference/kapt.html).


##### **Kotlin - KSP**

If you’re interested in using [ksp](https://kotlinlang.org/docs/ksp-overview.html), you can depend on Glide’s KSP support.

To do so, add the plugin:


```
apply plugin: 'com.google.devtools.ksp'
```


And depend on Glide’s snapshot KSP version:


```
ksp 'com.github.bumptech.glide:ksp:4.14.2'
```


**Note** - The KSP processor does not support Glide’s deprecated generated API. If you reference generated classes (`GlideApp`, `GlideRequests` etc), you will need to replace them with the non-generated equivalents before you can use KSP. See the [Generated API](https://bumptech.github.io/glide/doc/generatedapi.html) page for details.
