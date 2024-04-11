
##  **Configuration**



* [Setup](#setup)
    * [Applications](#applications)
    * [Libraries](#libraries)
        * [Avoid AppGlideModule in libraries](#avoid-appglidemodule-in-libraries)
    * [Setup Glide’s Annotation Processor](#setup-glides-annotation-processor)
* [Application Options](#application-options)
    * [Memory cache](#memory-cache)
    * [Bitmap pool](#bitmap-pool)
    * [Disk Cache](#disk-cache)
    * [Default Request Options](#default-request-options)
    * [UncaughtThrowableStrategy](#uncaughtthrowablestrategy)
    * [Log level](#log-level)
* [Registering Components](#registering-components)
    * [Anatomy of a load](#anatomy-of-a-load)
    * [Ordering Components](#ordering-components)
        * [prepend()](#prepend)
        * [append()](#append)
        * [replace()](#replace)
    * [Adding a ModelLoader](#adding-a-modelloader)
* [Module classes and annotations.](#module-classes-and-annotations)
    * [AppGlideModule](#appglidemodule)
    * [@GlideModule](#glidemodule)
    * [Annotation Processor](#annotation-processor)
* [Conflicts](#conflicts)
* [Manifest Parsing](#manifest-parsing)

### **Setup**


Starting in Glide 4.9.0, the setup described in this page is only required in a couple of cases.


For applications, setup is only required if the application wants to:

* Use one or more integration libraries
* Change Glide’s configuration (disk cache size/location, memory cache size etc)

For libraries, setup is only required if the library wants to register one or more components.


#### **Applications**


Applications that wish to use integration libraries and/or Glide’s API extensions must:

1. Add exactly one <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation
2. Optionally add one or more <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> implementations.
3. Add the <code>[@GlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideModule.html)</code> annotation to the <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation and all <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> implementations.
4. [Add a dependency on Glide’s annotation processor.](https://bumptech.github.io/glide/doc/download-setup.html#configuring-glide--annotation-processors)

An example <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> from Glide’s [Flickr sample app](https://github.com/bumptech/glide/blob/master/samples/flickr/src/main/java/com/bumptech/glide/samples/flickr/FlickrGlideModule.java) looks like this:



```
@GlideModule
public class FlickrGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(Photo.class, InputStream.class, new FlickrModelLoader.Factory());
  }
}
```



Including Glide’s annotation processor requires Glide’s annotation processor. See the [Download and Setup Page](https://bumptech.github.io/glide/doc/download-setup.html#configuring-glide--annotation-processors) for details on how to set up that dependency.


#### **Libraries**


Libraries that do not register custom components do not need to perform any configuration steps and can skip the sections on this page entirely.


Libraries that do need to register a custom component, like a `ModelLoader`, can do the following:



1. Add one or more <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> implementations that register the new components.
2. Add the <code>[@GlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideModule.html)</code> annotation to every <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> implementation.
3. [Add a dependency on Glide’s annotation processor.](https://bumptech.github.io/glide/doc/download-setup.html#configuring-glide--annotation-processors)

An example <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> from Glide’s [OkHttp integration library](https://github.com/bumptech/glide/blob/master/integration/okhttp3/src/main/java/com/bumptech/glide/integration/okhttp3/OkHttpLibraryGlideModule.java) looks like this:



```
@GlideModule
public final class OkHttpLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
  }
}
```



Using the <code>[@GlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideModule.html)</code> annotation requires a dependency on Glide’s annotations:


```
compile 'com.github.bumptech.glide:annotations:4.14.2'
```



#####   **Avoid AppGlideModule in libraries**


Libraries must **not** include `AppGlideModule` implementations. Doing so will prevent any applications that depend on the library from managing their dependencies or configuring options like Glide’s cache sizes and locations.


In addition, if two libraries include `AppGlideModule`s, applications will be unable to compile if they depend on both and will be forced to pick one or other other.


#### **Setup Glide’s Annotation Processor**


### **Application Options**


Glide allows applications to use <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementations to completely control Glide’s memory and disk cache usage. Glide tries to provide reasonable defaults for most applications, but for some applications, it will be necessary to customize these values. Be sure to measure the results of any changes to avoid performance regressions.


####  **Memory cache**

By default, Glide uses <code>[LruResourceCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/LruResourceCache.html)</code>, a default implementation of the <code>[MemoryCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html)</code> interface that uses a fixed amount of memory with LRU eviction. The size of the <code>[LruResourceCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/LruResourceCache.html)</code> is determined by Glide’s <code>[MemorySizeCalculator](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html)</code> class, which looks at the device memory class, whether or not the device is low ram and the screen resolution.


Applications can customize the <code>[MemoryCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html)</code> size in their <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> with the <code>[applyOptions(Context, GlideBuilder)](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html#applyOptions-android.content.Context-com.bumptech.glide.GlideBuilder-)</code> method by configuring <code>[MemorySizeCalculator](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html)</code>:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
        .setMemoryCacheScreens(2)
        .build();
    builder.setMemoryCache(new LruResourceCache(calculator.getMemoryCacheSize()));
  }
}
```



Applications can also directly override the cache size:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int memoryCacheSizeBytes = 1024 * 1024 * 20; // 20mb
    builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));
  }
}
```

Applications can even provide their own implementation of <code>[MemoryCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html)</code>:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new YourAppMemoryCacheImpl());
  }
}
```



#### **Bitmap pool**


Glide uses <code>[LruBitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/LruBitmapPool.html)</code> as the default <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code>. <code>[LruBitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/LruBitmapPool.html)</code> is a fixed size in memory <code>BitmapPool</code> that uses LRU eviction. The default size is based on the screen size and density of the device in question as well as the memory class and the return value of <code>[isLowRamDevice](https://developer.android.com/reference/android/app/ActivityManager.html#isLowRamDevice())</code>. The specific calculations are done by Glide’s <code>[MemorySizeCalculator](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html)</code>, similar to the way sizes are determined for Glide’s <code>[MemoryCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemoryCache.html)</code>.


Applications can customize the <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code> size in their <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> with the <code>[applyOptions(Context, GlideBuilder)](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html#applyOptions-android.content.Context-com.bumptech.glide.GlideBuilder-)</code> method by configuring <code>[MemorySizeCalculator](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html)</code>:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    MemorySizeCalculator calculator = new MemorySizeCalculator.Builder(context)
        .setBitmapPoolScreens(3)
        .build();
    builder.setBitmapPool(new LruBitmapPool(calculator.getBitmapPoolSize()));
  }
}
```



Applications can also directly override the pool size:
```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int bitmapPoolSizeBytes = 1024 * 1024 * 30; // 30mb
    builder.setBitmapPool(new LruBitmapPool(bitmapPoolSizeBytes));
  }
}
```


Applications can even provide their own implementation of <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code>:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setBitmapPool(new YourAppBitmapPoolImpl());
  }
}
```



#### **Disk Cache**

Glide uses <code>[DiskLruCacheWrapper](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html)</code> as the default <code>[DiskCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html)</code>. <code>[DiskLruCacheWrapper](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html)</code> is a fixed size disk cache with LRU eviction. The default disk cache size is [250 MB](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html#DEFAULT_DISK_CACHE_SIZE) and is placed in a [specific directory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html#DEFAULT_DISK_CACHE_DIR) in the Application’s [cache folder](https://developer.android.com/reference/android/content/Context.html#getCacheDir()).

Applications can change the location to external storage if the media they display is public (obtained from websites without authentication, search engines etc):


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDiskCache(new ExternalCacheDiskCacheFactory(context));
  }
}
```


Applications can change the size of the disk, for either the internal or external disk caches:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int diskCacheSizeBytes = 1024 * 1024 * 100; // 100 MB
    builder.setDiskCache(new InternalCacheDiskCacheFactory(context, diskCacheSizeBytes));
  }
}
```

Applications can change the name of the folder the cache is placed in within external or internal storage:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    int diskCacheSizeBytes = 1024 * 1024 * 100; // 100 MB
    builder.setDiskCache(
        new InternalCacheDiskCacheFactory(context, "cacheFolderName", diskCacheSizeBytes));
  }
}
```

Applications can also choose to implement the <code>[DiskCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html)</code> interface themselves and provide their own <code>[DiskCache.Factory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html)</code> to produce it. Glide uses a Factory interface to open <code>[DiskCaches](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html)</code> on background threads so that the caches can do I/O like checking the existence of their target directory without causing a [StrictMode](https://developer.android.com/reference/android/os/StrictMode.html) violation.


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDiskCache(new DiskCache.Factory() {
        @Override
        public DiskCache build() {
          return new YourAppCustomDiskCache();
        }
    });
  }
}
```



#### **Default Request Options**

Although <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html)</code> are typically specified per request, you can also apply a default set of <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html)</code> that will be applied to every load you start in your application by using an <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDefaultRequestOptions(
        new RequestOptions()
          .format(DecodeFormat.RGB_565)
          .disallowHardwareBitmaps());
  }
}
```

 Options applied with `setDefaultRequestOptions` in `GlideBuilder` are applied as soon as you create a new request. As a result, options applied to any individual request will override any conflicting options that are set in the `GlideBuilder`.


<code>[RequestManagers](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html)</code> similarly allow you to set default <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html)</code> for all loads started with that particular <code>[RequestManager](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html)</code>. Since each <code>Activity</code> and <code>Fragment</code> gets its own <code>[RequestManager](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html)</code>, you can use <code>[RequestManager's](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html)</code> <code>[applyDefaultRequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html#applyDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-)</code> method to set default <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html)</code> that apply only to a particular <code>Activity</code> or <code>Fragment</code>:


```
Glide.with(fragment)
  .applyDefaultRequestOptions(
      new RequestOptions()
          .format(DecodeFormat.RGB_565)
          .disallowHardwareBitmaps());
```



<code>[RequestManager](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html)</code> also has a <code>[setDefaultRequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html#setDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-)</code> that will completely replace any default <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html)</code> previously set either via the <code>GlideBuilder</code> in an <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> or via the <code>[RequestManager](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html)</code>. Use caution with <code>[setDefaultRequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html#setDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-)</code> because it’s easy to accidentally override important defaults you’ve set elsewhere. Typically <code>[applyDefaultRequestOptions](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/RequestManager.html#applyDefaultRequestOptions-com.bumptech.glide.request.RequestOptions-)</code> is safer and more intuitive to use.


#### **UncaughtThrowableStrategy**

When loading a bitmap, if an exception happens (e.g. `OutOfMemoryException`), Glide will use a `GlideExecutor.UncaughtThrowableStrategy`. The default strategy is to log the exception in the device logcat. The strategy is customizable since Glide 4.2.0. It can be passed to a disk executor and/or a resize executor:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    final UncaughtThrowableStrategy myUncaughtThrowableStrategy = new ...
    builder.setDiskCacheExecutor(newDiskCacheExecutor(myUncaughtThrowableStrategy));
    builder.setResizeExecutor(newSourceExecutor(myUncaughtThrowableStrategy));
  }
}
```



#### **Log level**

For a subset of well formatted logs, including lines logged when a request fails, you can use <code>[setLogLevel](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/GlideBuilder.html#setLogLevel-int-)</code> with one of the values from Android’s <code>[Log](https://developer.android.com/reference/android/util/Log.html)</code> class. Generally speaking <code>log.VERBOSE</code> will make logs noisier and <code>Log.ERROR</code> will make logs quieter, but see [the javadoc](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/GlideBuilder.html#setLogLevel-int-) for details.


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setLogLevel(Log.DEBUG);
  }
}
```



###  **Registering Components**

 Both Applications and Libraries can register a number of components that extend Glides functionality. Available components include:



1. <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/ModelLoaderFactory.html)</code>s to load custom Models (Urls, Uris, arbitrary POJOs) and Data (InputStreams, FileDescriptors).
2. <code>[ResourceDecoder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html)</code>s to decode new Resources (Drawables, Bitmaps) or new types of Data (InputStreams, FileDescriptors).
3. <code>[Encoder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Encoder.html)</code>s to write Data (InputStreams, FileDescriptors) to Glide’s disk cache.
4. <code>[ResourceTranscoder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/transcode/ResourceTranscoder.html)</code>s to convert Resources (BitmapResource) into other types of Resources (DrawableResource).
5. <code>[ResourceEncoder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/ResourceEncoder.html)</code>s to write Resources (BitmapResource, DrawableResource) to Glide’s disk cache.

 Components are registered using the <code>[Registry](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/Registry.html)</code> class in the <code>[registerComponents()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html#registerComponents-android.content.Context-com.bumptech.glide.Glide-com.bumptech.glide.Registry-)</code> method of <code>AppGlideModules</code> and <code>LibraryGlideModules</code>:



```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(...);
  }
}
```



 or:


```
@GlideModule
public class YourLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(...);
  }
}
```


Any number of components can registered in a single `GlideModule`. Certain types, including <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/ModelLoaderFactory.html)</code>s and <code>[ResourceDecoder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html)</code>s can have multiple implementations with the same type arguments.


#### **Anatomy of a load**

The set of registered components, including both those registered by default in Glide and those registered in Modules are used to define a set of load paths. Each load path is a step by step progression from the the Model provided to <code>[load()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#load-java.lang.Object-)</code> to the Resource type specified by <code>[as()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html#as-java.lang.Class-)</code>. A load path consists (roughly) of the following steps:



1. Model -> Data (handled by `ModelLoader`s)
2. Data -> Resource (handled by `ResourceDecoder`s)
3. Resource -> Transcoded Resource (optional, handled by `ResourceTranscoder`s).

`Encoder`s can write Data to Glide’s disk cache cache before step 2. `ResourceEncoder`s can write Resource’s to Glide’s disk cache before step 3.


 When a request is started, Glide will attempt all available paths from the Model to the requested Resource type. A request will succeed if any load path succeeds. A request will fail only if all available load paths fail.


####  **Ordering Components**

The `prepend()`, `append()`, and `replace()` methods in <code>[Registry](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/Registry.html)</code> can be used to set the order in which Glide will attempt each <code>ModelLoader</code> and <code>ResourceDecoder</code>. Ordering components allows you to register components that handle specific subsets of models or data (IE only certain types of Uris, or only certain image formats) while also having an appended catch-all component to handle the rest.


##### **prepend()**

To handle subsets of existing data where you do want to fall back to Glide’s default behavior if your `ModelLoader` or `ResourceDecoder` fails, using `prepend()`. `prepend()` will make sure that your `ModelLoader` or `ResourceDecoder` is called before all other previously registered components and can run first. If your `ModelLoader` or `ResourceDecoder` returns `false` from its `handles()` method or fails, all other `ModelLoader`s or `ResourceDecoders` will be called in the order they’re registered, one at a time, providing a fallback.


##### **append()**

To handle new types of data or to add a fallback to Glide’s default behavior, using `append()`. `append()` will make sure that your `ModelLoader` or `ResourceDecoder` is called only after Glide’s defaults are attempted. If you’re trying to handle subtypes that Glide’s default components handle (like a specific Uri authority or subclass), you may need to use `prepend()` to make sure Glide’s default component doesn’t load the resource before your custom component.


##### **replace()**

To completely replace Glide’s default behavior and ensure that it doesn’t run, use `replace()`. `replace()` removes all `ModelLoaders` that handle the given model and data classes and then adds your `ModelLoader` instead. `replace()` is useful in particular for swapping out Glide’s networking logic with a library like OkHttp or Volley, where you want to make sure that only OkHttp or Volley are used.


#### **Adding a ModelLoader**

For example, to add a `ModelLoader` that can obtain an InputStream for a new custom Model object:



```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(Photo.class, InputStream.class, new CustomModelLoader.Factory());
  }
}
```


 `append()` can be used safely here because Photo.class is a custom model object specific to your application, so you know that there is no default behavior in Glide that you need to replace.

In contrast, to add handling for a new type of String url in a <code>[BaseGlideUrlLoader](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/stream/BaseGlideUrlLoader.html)</code>, you should use <code>prepend()</code> so that your <code>ModelLoader</code> gets to run before Glide’s default <code>ModelLoaders</code> for <code>Strings</code>:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.prepend(String.class, InputStream.class, new CustomUrlModelLoader.Factory());
  }
}
```



Finally to completely remove and replace Glide’s default handling of a certain type, like a networking library, you should use `replace()`:


```
@GlideModule
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new OkHttpUrlLoader.Factory());
  }
}
```



### **Module classes and annotations.**

Glide v4 relies on two classes, <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> and <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code>, to configure the Glide singleton. Both classes are allowed to register additional components, like <code>[ModelLoaders](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/ModelLoader.html)</code>, <code>[ResourceDecoders](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html)</code> etc. Only the <code>[AppGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> are allowed to configure application specific settings, like cache implementations and sizes.


#### **AppGlideModule**

All applications can optionally add a <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation if the Application is implementing any methods in <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> or wants to use any integration libraries. The <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation acts as a signal that allows Glide’s annotation processor to generate a single combined class with with all discovered <code>[LibraryGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code>.


There can be only one <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation in a given application (having more than one produce errors at compile time). As a result, libraries must never provide a <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation.


#### **@GlideModule**

In order for Glide to properly discover <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> and <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> implementations, all implementations of both classes must be annotated with the <code>[@GlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/GlideModule.html)</code> annotation. The annotation will allow Glide’s [annotation processor](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/compiler/GlideAnnotationProcessor.html) to discover all implementations at compile time.


#### **Annotation Processor**

In addition, to enable discovery of the <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> and <code>[LibraryGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> all libraries and applications must also include a dependency on Glide’s annotation processor.


### **Conflicts**

Applications may depend on multiple libraries, each of which may contain one or more <code>[LibraryGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code>. In rare cases, these <code>[LibraryGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> may define conflicting options or otherwise include behavior the application would like to avoid. Applications can resolve these conflicts or avoid unwanted dependencies by adding an <code>[@Excludes](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/Excludes.html)</code> annotation to their <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>.


For example if you depend on a library that has a <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> that you’d like to avoid, say <code>com.example.unwanted.GlideModule</code>:


```
@Excludes(com.example.unwanted.GlideModule.class)
@GlideModule
public final class MyAppGlideModule extends AppGlideModule { }
```



You can also excludes multiple modules:


```
@Excludes({com.example.unwanted.GlideModule.class, com.example.conflicing.GlideModule.class})
@GlideModule
public final class MyAppGlideModule extends AppGlideModule { }
```



 <code>[@Excludes](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/annotation/Excludes.html)</code> can be used to exclude both <code>[LibraryGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> and legacy, deprecated <code>[GlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/GlideModule.html)</code> implementations if you’re still in the process of migrating from Glide v3.


### **Manifest Parsing**

To maintain backward compatibility with Glide v3’s <code>[GlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/GlideModule.html)</code>, Glide still parses <code>AndroidManifest.xml</code> files from both the application and any included libraries and will include any legacy <code>[GlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/GlideModule.html)</code> listed in the manifest. Although this functionality will be removed in a future version, we’ve retained the behavior for now to ease the transition.


If you’ve already migrated to the Glide v4 <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> and <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code>, you can disable manifest parsing entirely. Doing so can improve the initial startup time of Glide and avoid some potential problems with trying to parse metadata. To disable manifest parsing, override the <code>[isManifestParsingEnabled()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html#isManifestParsingEnabled--)</code> method in your <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation:


```
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {
  @Override
  public boolean isManifestParsingEnabled() {
    return false;
  }
}
