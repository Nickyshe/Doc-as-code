
## **Generated API**



* [This page and the generated API are deprecated](#this-page-and-the-generated-api-are-deprecated)
    * [About](#about)
    * [Getting Started](#getting-started)
        * [Availability](#availability)
        * [Java](#java)
        * [Kotlin](#kotlin)
        * [Android Studio](#android-studio)
    * [Using the generated API](#using-the-generated-api)
    * [GlideExtension](#glideextension)
        * [GlideOption](#glideoption)
        * [GlideType](#glidetype)

## **This page and the generated API are deprecated**


The generated API is deprecated as of Glide 4.14.0. Glide’s annotation processors will continue to be used for [configuration](https://nickyshe.github.io/Glide-V4/#/Configurations).


Specifically:

1. You should not add new <code>[Extensions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code> as described on this page
2. You should not use generated classes. Use the equivalent non-generated classes instead:
    * Instead of <code>GlideApp</code>, use <code>com.bumptech.Glide</code>
    * Instead of <code>GlideRequests</code>, use <code>com.bumptech.glide.RequestManager</code>
    * Instead of <code>GlideRequest</code>, use <code>com.bumptech.glide.RequestBuilder</code>
    * Instead of <code>GlideOptions</code>, use <code>com.bumptech.glide.request.RequestOptions</code>

    You should continue to use the annotation processors and <code>AppGlideModule</code> or <code>LibraryGlideModule</code> classes as necessary to [configure Glide](https://nickyshe.github.io/Glide-V4/#/Configurations). Configuration using these classes and annotation processing is <em>NOT</em> deprecated.


    The generated API is deprecated because:

1. Glide 4.9.0 integrated RequestOptions into RequestBuilder using inheritance instead of code gen.
2. `Extensions`, the other functionality added by this API, seem to be rarely used.
3. `Extensions` can be trivially replicated in Kotlin using extension functions with no additional support from Glide.
4. Generating the classes (`GlideApp`, `GlideRequests` etc) adds build time and complexity in Glide’s annotation processors.

In practice there’s no plan to remove support for the generated API from Glide’s Java based annotation processor. However we do not plan to add support for the generated API to Glide’s KSP integration or add additional related functionality in the future.


### **About**


Glide v4 uses an [annotation processor](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/Processor.html) to generate an API that allows applications to extend Glide’s API and include components provided by integration libraries.


The generated API serves two purposes:

1. Integration libraries can extend Glide’s API with custom options.
2. Applications can extend Glide’s API by adding methods that bundle commonly used options.

Glide originally added a generated API that made it easy to extend RequestOptions. Since that’s now trivial to do with extension functions in Kotlin, the API extensions should be considered deprecated. The configuration options, including LibraryGlideModule and AppGlideModule are not deprecated and will continue to be used. Only the extensions mentioned on this page are targeted for eventual removal.


### **Getting Started**


#### **Availability**

The generated API is only available for applications.


The API is only generated when a properly annotated `AppGlideModule` is found. There can only be one `AppGlideModule` per application. As a result it’s not possible to generate the API for a library without precluding any application that uses the library from using the generated API.


#### **Java**


To use the generated API in your application, you need to perform two steps:

1. [Add a dependency on Glide’s annotation processor](https://bumptech.github.io/glide/doc/download-setup.html)

Include a <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation in your application: \
<strong><code>package com.example.myapp;</code></strong>


```
import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}

```



2.  You’re not required to implement any of the methods in `AppGlideModule` for the API to be generated. You can leave the class blank as long as it extends `AppGlideModule` and is annotated with `@GlideModule`.


<code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementations must always be annotated with <code>[@GlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideModule.html)</code>. If the annotation is not present, the module will not be discovered and you will see a warning in your logs with the <code>Glide</code> log tag that indicates that the module couldn’t be found.


**Note:** Libraries should **not** include <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementations. See the [configuration](https://nickyshe.github.io/Glide-V4/#/Configurations) page for details.


####  **Kotlin**


If you’re using Kotlin you can:

1. [Add a dependency on Glide’s annotation processor](https://nickyshe.github.io/Glide-V4/#/Download_Setup)
2. Implement all of Glide’s annotated classes (<code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>, <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code>, and <code>[GlideExtension](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code>) in Java as shown above.

#### <strong>Android Studio</strong>

For the most part Android Studio just works with annotation processors and the generated API. However, you may need to rebuild the project the first time you add your `AppGlideModule` or after some types of changes. If the API won’t import or seems out of date, you can re-build by:

1. Open the Build menu
2. Click Rebuild Project.

### **Using the generated API**


**This API is deprecated.**


 The API is generated in the same package as the <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation provided by the application and is named <code>GlideApp</code> by default. Applications can use the API by starting all loads with <code>GlideApp.with()</code> instead of <code>Glide.with()</code>:



```
GlideApp.with(fragment)
   .load(myUrl)
   .placeholder(R.drawable.placeholder)
   .fitCenter()
   .into(imageView);
```


Unlike `Glide.with()` options like `fitCenter()` and `placeholder()` are available directly on the builder and don’t need to be passed in as a separate <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code> object.


### **GlideExtension**


**This API is deprecated.**


Glide’s generated API can be extended by both Applications and Libraries. Extensions use annotated static methods to add new options, modifying existing options, or add additional types.


The <code>[GlideExtension](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code> annotation identifies a class that extends Glide’s API. The annotation must be present on any classes that extend Glide’s API. If the annotation is not present, annotations on methods will be ignored.


Classes annotated with <code>[GlideExtension](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code> are expected to be utility classes. They should have a private and empty constructor. Classes annotated with GlideExtension should also be final and contain only static methods. Annotated classes may contain static variables and may reference other classes or objects.


An application may implement as many <code>[GlideExtension](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code> annotated classes as they’d like. Libraries can also implement an arbitrary number of <code>[GlideExtension](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code> annotated classes. When a <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> is found, all available <code>[GlideExtensions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code> will be merged to create a single API with all available extensions. Conflicts will result in compilation errors in Glide’s annotation processor.


GlideExtension annotated classes can define two types of extension methods:



1. <code>[GlideOption](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideOption.html)</code> - Adds a custom option to <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code>.
2. <code>[GlideType](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideType.html)</code> - Adds support for a new resource type (GIFs, SVG etc).

####  <strong>GlideOption</strong>

**This API is deprecated.**

<code>[GlideOption](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideOption.html)</code> annotated static methods extend <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code>. <code>GlideOption</code> is useful to:

1. Define a group of options that is used frequently throughout an application.
2. Add new options, typically in conjunction with Glide’s <code>[Option](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Option.html)</code> class.

To define a group of options, you might write:



```
@GlideExtension
public class MyAppExtension {
  // Size of mini thumb in pixels.
  private static final int MINI_THUMB_SIZE = 100;

  private MyAppExtension() { } // utility class

  @NonNull
  @GlideOption
  public static BaseRequestOptions<?> miniThumb(BaseRequestOptions<?> options) {
    return options
      .fitCenter()
      .override(MINI_THUMB_SIZE);
  }
```



This will generate a method in a <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code> subclass that looks like this:


```
public class GlideOptions extends RequestOptions {


  public GlideOptions miniThumb() {
    return (GlideOptions) MyAppExtension.miniThumb(this);
  }

  ...
}
```


You can include as many additional arguments in your methods as you want, as long as the first argument is always <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code>:


```
@GlideOption
public static BaseRequestOptions<?> miniThumb(BaseRequestOptions<?> options, int size) {
  return options
    .fitCenter()
    .override(size);
}
```

The additional arguments will be added as arguments to the generated method:


```
public GlideOptions miniThumb(int size) {
  return (GlideOptions) MyAppExtension.miniThumb(this);
}
```

You can then call your custom method by using the generated `GlideApp` class:


```
GlideApp.with(fragment)
   .load(url)
   .miniThumb(thumbnailSize)
   .into(imageView);
```


Methods with the `GlideOption` annotation are expected to be static and to return `BaseRequestOptions&lt;?>`. Note that the generated methods will not be available on the standard `Glide` and `RequestOptions` classes, only the generated equivalents.


#### **GlideType**

**This API is deprecated.**


<code>[GlideType](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideType.html)</code> annotated static methods extend <code>[RequestManager](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html)</code>. <code>GlideType</code> annotated methods allow you to add support for new types, including specifying default options.


For example, to add support for GIFs, you might add a `GlideType` method:


```
@GlideExtension
public class MyAppExtension {
  private static final RequestOptions DECODE_TYPE_GIF = decodeTypeOf(GifDrawable.class).lock();

  @NonNull
  @GlideType(GifDrawable.class)
  public static RequestBuilder<GifDrwable> asGif(RequestBuilder<GifDrawable> requestBuilder) {
    return requestBuilder
      .transition(new DrawableTransitionOptions())
      .apply(DECODE_TYPE_GIF);
  }
}
```



Doing so will generate a <code>[RequestManager](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html)</code> with a method that looks like this:


```
public class GlideRequests extends RequesetManager {

  public GlideRequest<GifDrawable> asGif() {
    return (GlideRequest<GifDrawable> MyAppExtension.asGif(this.as(GifDrawable.class));
  }


  ...
}
```


You can then use the generated `GlideApp` class to call your custom type:


```
GlideApp.with(fragment)
  .asGif()
  .load(url)
  .into(imageView);
```


Methods annotated with `GlideType` must take a <code>[RequestBuilder&lt;T>](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code> as their first argument where the type <code>&lt;T></code> matches the class provided to the <code>[GlideType](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideType.html)</code> annotation. Methods are expected to be static and return <code>RequestBuilder&lt;T></code>. Methods must be defined in a class annotated with <code>[GlideExtension](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideExtension.html)</code>.
