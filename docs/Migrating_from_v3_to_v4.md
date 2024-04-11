

## **Migrating from v3 to v4**



* [Options](#options)
    * [RequestBuilder](#requestbuilder)
    * [RequestOptions](#requestoptions)
    * [Transformations](#transformations)
    * [DecodeFormat](#decodeformat)
    * [TransitionOptions](#transitionoptions)
        * [Cross fades.](#cross-fades)
    * [Generated API](#generated-api)
* [Types and Targets](#types-and-targets)
    * [Picking Resource Types](#picking-resource-types)
    * [Drawables](#drawables)
    * [Targets](#targets)
        * [Cancellation](#cancellation)
* [Configuration](#configuration)
    * [Applications](#applications)
    * [Libraries](#libraries)
    * [Manifest parsing](#manifest-parsing)
    * <code>[using(), ModelLoader, StreamModelLoader.](#using-modelloader-streammodelloader)</code>
        * [ModelLoader](#modelloader)</code>

## <strong>Options</strong>


One of the larger changes in Glide v4 is the way the library handles options (`centerCrop()`, `placeholder()` etc). In Glide v3, options were handled individually by a series of complicated multityped builders. In Glide v4 these have been replaced by a single builder with a single type and a series of options objects that can be provided to the builder. Glide’s [generated API](https://bumptech.github.io/glide/doc/generatedapi.html) simplifies this further by merging options from the options objects and from any included integration libraries with the builder to create a single fluent API.


### **RequestBuilder**


 Includes methods like:



```
listener()
thumbnail()
load()
into()
```

In Glide v4 there is only a single <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code> with a single type that indicates the type of item you’re attempting to load (<code>Bitmap</code>, <code>Drawable</code>, <code>GifDrawable</code> etc). The <code>RequestBuilder</code> provides direct access to options that affect the load process itself, including the model (url, uri etc) you want to load, any <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> requests and any <code>[RequestListeners](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#listener-com.bumptech.glide.request.RequestListener-)</code>. The <code>RequestBuilder</code> also is the place where you start the load using <code>[into()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-Y-)</code> or <code>[preload()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#preload-int-int-)</code>:


```
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment)
    .load(url);

requestBuilder
    .thumbnail(Glide.with(fragment)
        .load(thumbnailUrl))
    .listener(requestListener)
    .load(url)
    .into(imageView);
```



### **RequestOptions**


Includes methods like:


```
centerCrop()
placeholder()
error()
priority()
diskCacheStrategy()
```



Most options have moved into a separate object called <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code>:


```
RequestOptions options = new RequestOptions()
    .centerCrop()
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.error)
    .priority(Priority.HIGH);
```



    `RequestOptions` are applied to `RequestBuilders` to allow you to specify a set of options once and then use them for multiple loads:


```
RequestOptions myOptions = new RequestOptions()
    .fitCenter()
    .override(100, 100);

Glide.with(fragment)
    .load(url)
    .apply(myOptions)
    .into(drawableView);

Glide.with(fragment)
    .asBitmap()
    .apply(myOptions)
    .load(url)
    .into(bitmapView);
```



### **Transformations**


 <code>[Transformations](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/load/Transformation.html)</code> in Glide v4 now replace any previously set transformations. If you want to apply more than one <code>[Transformation](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/load/Transformation.html)</code> in Glide v4, use the <code>[transforms()](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html#transforms-com.bumptech.glide.load.Transformation...-)</code> method:


```
Glide.with(fragment)
  .load(url)
  .transforms(new CenterCrop(), new RoundedCorners(20))
  .into(target);
```



### **DecodeFormat**


In Glide v3, the default <code>[DecodeFormat](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/load/DecodeFormat.html)</code> was <code>[DecodeFormat.PREFER_RGB_565](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/load/DecodeFormat.html#PREFER_RGB_565)</code>, which used <code>[Bitmap.Config.RGB_565](https://developer.android.com/reference/android/graphics/Bitmap.Config.html#RGB_565)</code> unless the image contained or might have contained transparent pixels. <code>RGB_565</code> uses half the memory of <code>[Bitmap.Config.ARGB_8888](https://developer.android.com/reference/android/graphics/Bitmap.Config.html#ARGB_8888)</code> for a given image size, but it has noticeable quality issues for certain images, including banding and tinting. To avoid the quality issues with <code>RGB_565</code>, Glide defaults to <code>ARGB_8888</code>. As a result, image quality is higher, but memory usage may increase.


To change Glide’s default <code>[DecodeFormat](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/load/DecodeFormat.html)</code> back to <code>[DecodeFormat.PREFER_RGB_565](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/load/DecodeFormat.html#PREFER_RGB_565)</code> in Glide v4, apply the <code>RequestOption</code> in an <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>:


```
@GlideModule
public final class YourAppGlideModule extends GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setDefaultRequestOptions(new RequestOptions().format(DecodeFormat.PREFER_RGB_565));
  }
}
```


For more on using <code>[AppGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>, see the [configuration page](https://bumptech.github.io/glide/doc/configuration.html). Note that you will have to make sure to add a dependency on Glide’s annotation processor to ensure that Glide picks up your <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation. For more information on how to set up the library, see the [download and setup page](https://bumptech.github.io/glide/doc/download-setup.html).


###  **TransitionOptions**


Includes methods like:


```
crossFade()
animate()
```



Options that control cross fades and other transitions from placeholders to images and/or between thumbnails and the full image have been moved into <code>[TransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html)</code>.


To apply transitions (formerly animations), use one of the transition options that matches the type of resource you’re requesting:



* <code>[GenericTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/GenericTransitionOptions.html)</code>
* <code>[DrawableTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html)</code>
* <code>[BitmapTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html)</code>

To remove any default transition, use <code>[TransitionOptions.dontTransition()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/TransitionOptions.html#dontTransition--)</code>.


Transitions are applied to a request using <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code>:



```
Glide.with(fragment)
    .load(url)
    .transition(withCrossFade(R.anim.fade_in, 300));
```



#### **Cross fades.**


Unlike Glide v3, Glide v4 does **NOT** apply a cross fade or any other transition by default to requests. Transitions must be applied manually.


To apply a cross fade transition to a particular load, you can use:


```
import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;

Glide.with(fragment)
  .load(url)
  .transition(withCrossFade())
  .into(imageView);
```



Or:


```
Glide.with(fragment)
  .load(url)
  .transition(
      new DrawableTransitionOptions
        .crossFade())
  .into(imageView);
```



### **Generated API**


 To make it even easier to use Glide v4, Glide now also offers a generated API for Applications. Applications can access the generated API by including an appropriately annotated <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> implementation. See the [Generated API](https://bumptech.github.io/glide/doc/generatedapi.html) page for details on how this works.


You can still use the generated `RequestOptions` subclass to apply the same set of options to multiple loads, but generated `RequestBuilder` subclass may be more convenient in most cases.


## **Types and Targets**


### **Picking Resource Types**

Glide allows you to specify what type of resource you want to load. If you specify a super type, Glide will attempt to load any available subtypes. For example, if you request a Drawable, Glide may load either a BitmapDrawable or a GifDrawable. If you request a GifDrawable, Glide will either load a GifDrawable or error if the image isn’t a GIF (even if it happens to be a perfectly valid image).


Drawables are requested by default:


```
Glide.with(fragment).load(url)
```



To request a Bitmap:


```
Glide.with(fragment).asBitmap()
```



To obtain a filepath (best for local images):


```
Glide.with(fragment).asFile()
```



To download a remote file into cache and obtain a file path:


```
Glide.with(fragment).downloadOnly()
// or if you have the url already:
Glide.with(fragment).download(url);
```



### **Drawables**
`GlideDrawable` in Glide v3 has been removed in favor of the standard Android <code>[Drawable](https://developer.android.com/reference/android/graphics/drawable/Drawable.html)</code>. <code>GlideBitmapDrawable</code> has been removed in favor of <code>[BitmapDrawable](https://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html)</code>.


If you want to know if a Drawable is animated, you can check if it is an instance of <code>[Animatable](https://developer.android.com/reference/android/graphics/drawable/Animatable.html)</code>:


```
boolean isAnimated = drawable instanceof Animatable
```



###  **Targets**


The signature of `onResourceReady` has changed. For example, for `Drawables`:


```
onResourceReady(GlideDrawable drawable, GlideAnimation<? super GlideDrawable> anim) 
```



is now:


```
onResourceReady(Drawable drawable, Transition<? super Drawable> transition);
```


 Similarly the signature of `onLoadFailed` has also changed:


```
onLoadFailed(Exception e, Drawable errorDrawable)
```



is now:


```
onLoadFailed(Drawable errorDrawable)
```

 If you need more information about the errors that caused the load to fail, you can use <code>[RequestListener](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestListener.html)</code>.


#### **Cancellation**

`Glide.clear(Target)` has moved into <code>[RequestManager](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html)</code>:


```
Glide.with(fragment).clear(target)
```


Although it’s not required, it’s most performant to use the `RequestManager` that started the load to also clear the load. Glide v4 keeps track of requests per Activity and Fragment so clearing needs to remove the request at the appropriate level.


##  **Configuration**

In Glide v3, configuration is performed via one or more <code>[GlideModules](https://bumptech.github.io/glide/javadocs/360/com/bumptech/glide/module/GlideModule.html)</code>. In Glide v4, configuration is done via a similar but slightly more sophisticated system.


For details on the new system, see the [Configuration](https://bumptech.github.io/glide/doc/configuration.html) page.


### **Applications**

Applications that have a single <code>[GlideModule](https://bumptech.github.io/glide/javadocs/360/com/bumptech/glide/module/GlideModule.html)</code> can convert their <code>GlideModule</code> into a <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>.


In Glide v3, you might have a `GlideModule` like this:


```
public class GiphyGlideModule implements GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new LruResourceCache(10 * 1024 * 1024));
  }

  @Override
  public void registerComponents(Context context, Glide glide) {
    glide.register(Api.GifResult.class, InputStream.class, new GiphyModelLoader.Factory());
  }
}
```

In Glide v4, you would convert it into a `AppGlideModule` that looks like this:


```
@GlideModule
public class GiphyGlideModule extends AppGlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    builder.setMemoryCache(new LruResourceCache(10 * 1024 * 1024));
  }

  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(Api.GifResult.class, InputStream.class, new GiphyModelLoader.Factory());
  }
}
```

Note that the `@GlideModule` annotation is required.


If your application has multiple `GlideModule`s, convert one of them to a `AppGlideModule` and the others to <code>[LibraryGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code>. <code>LibraryGlideModule</code>s will not be discovered unless a <code>AppGlideModule</code> is present, so you cannot use only <code>LibraryGlideModule</code>s.


### **Libraries**

Libraries that have one or more `GlideModule`s should use <code>[LibraryGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/LibraryGlideModule.html)</code> instead of <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code>. Libraries should not use <code>[AppGlideModules](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> because there can only be one per Application, so including it in a library would not only prevent users of the library from setting their own options, but it would also cause conflicts if multiple libraries included a <code>AppGlideModule</code>.


For example, the Volley `GlideModule` in v3:


```
public class VolleyGlideModule implements GlideModule {
  @Override
  public void applyOptions(Context context, GlideBuilder builder) {
    // Do nothing.
  }

  @Override
  public void registerComponents(Context context, Glide glide) {
    glide.register(GlideUrl.class, InputStream.class, new VolleyUrlLoader.Factory(context));
  }
}
```


Can be converted to a `LibraryGlideModule` in v4:


```
@GlideModule
public class VolleyLibraryGlideModule extends LibraryGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.replace(GlideUrl.class, InputStream.class, new VolleyUrlLoader.Factory(context));
  }
}
```



### **Manifest parsing**

To ease the migration, manifest parsing and the older <code>[GlideModule](https://bumptech.github.io/glide/javadocs/360/com/bumptech/glide/module/GlideModule.html)</code> interface are deprecated, but still supported in v4. <code>AppGlideModule</code>s, <code>LibraryGlideModule</code>s and the deprecated <code>GlideModule</code>s can all coexist in an application.


However, to avoid the performance overhead of checking metadata (and associated bugs), you can disable manifest parsing once your migration is complete by overriding a method in your `AppGlideModule`:


```
@GlideModule
public class GiphyGlideModule extends AppGlideModule {
  @Override
  public boolean isManifestParsingEnabled() {
    return false;
  }

  ...
}
```



### **<code>using()</code>, ModelLoader, StreamModelLoader.</strong>**


#### **ModelLoader**


The <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/ModelLoader.html)</code> API exists in Glide v4 and serves the same purpose that it did in Glide v3, but a few of the specifics have changed.


First specific subtypes of `ModelLoader`, like `StreamModelLoader` are now unnecessary and users can implement `ModelLoader` directly. For example, a `StreamModelLoader&lt;File>` would now be implemented and referred to as a `ModelLoader&lt;File, InputStream>`.

 Second, instead of returning a `DataFetcher` directly, `ModelLoader`s now return <code>[LoadData](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/ModelLoader.LoadData.html)</code>. <code>LoadData</code> is a very simple wrapper that contains a disk cache key and a <code>DataFetcher</code>.


Third, `ModelLoaders` have a `handles()` method, so that you can register more than one ModelLoader with the same type parameters.


Converting a `ModelLoader` from the v3 API to the v4 API is almost always straight forward. If you just return a `DataFetcher` in your v3 `ModelLoader`:


```
public final class MyModelLoader implements StreamModelLoader<File> {

  @Override
  public DataFetcher<InputStream> getResourceFetcher(File model, int width, int height) {
    return new MyDataFetcher(model);
  }
}
```


Then all you need to do in your v4 equivalent is wrap the data fetcher:


```
public final class MyModelLoader implements ModelLoader<File, InputStream> {

  @Override
  public LoadData<InputStream> buildLoadData(File model, int width, int height,
      Options options) {
    return new LoadData<>(model, new MyDataFetcher(model));
  }

  @Override
  public void handles(File model) {
    return true;
  }
}
```



Note that the model is passed in to the `LoadData` to act as part of the cache key, in addition to the `DataFetcher`. This pattern provides more control over the disk cache key in some specialized circumstances. Most implementations can just pass their model directly into `LoadData` as is done above. For this to work correctly your model needs to correctly implements `hashCode()` and `equals()`


If you’d only like to use your ModelLoader for some models you can use the `handles()` method to inspect the model before you try to load it. If you return `false` from `handles()` your `ModelLoader` will not be to load the given model, even if the types of your `ModelLoader` (`File` and `InputStream` in this example) match.


For example, if you’re writing encrypted images to disk in a specific folder, you could use the `handles()` method to implement a `ModelLoader` that decrypted images from that specific folder but wasn’t used when loading `File`s from other folders:


```
public final class MyModelLoader implements ModelLoader<File, InputStream> {
  private static final String ENCRYPTED_PATH = "/my/encrypted/folder";

  @Override
  public LoadData<InputStream> buildLoadData(File model, int width, int height,
      Options options) {
    return new LoadData<>(model, new MyDataFetcher(model));
  }

  @Override
  public void handles(File model) {
    return model.getAbsolutePath().startsWith(ENCRYPTED_PATH);
  }
}

```
#### **using()**


The <code>[using()](https://bumptech.github.io/glide/javadocs/380/com/bumptech/glide/RequestManager.html#using(com.bumptech.glide.load.model.stream.StreamByteArrayLoader))</code> API was removed in Glide 4 to encourage users to [register](https://bumptech.github.io/glide/doc/configuration.html#registering-components) their components once with a <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> to avoid object re-use. Rather than creating a new <code>ModelLoader</code> each time you load an image, you register it once in an <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/module/AppGlideModule.html)</code> and let Glide inspect your model (the object you pass to <code>[load()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#load-java.lang.Object-)</code>) to figure out when to use your registered <code>ModelLoader</code>.


To make sure you only use your `ModelLoader` for certain models, implement `handles()` as shown above to inspect each model and return true only if your `ModelLoader` should be used.
