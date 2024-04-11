

##  **Writing a custom ModelLoader**



* [Writing the ModelLoader.](#writing-a-custom-modelloader)
    * [An empty implementation](#an-empty-implementation)
    * [Implementing handles()](#implementing-handles)
    * [Implementing buildLoadData](#implementing-buildloaddata)
        * [Picking the Key](#picking-the-key)
        * [Picking the DataFetcher](#picking-the-datafetcher)
* [Writing the DataFetcher](#writing-the-datafetcher)
    * [getDataClass](#getdataclass)
    * [getDataSource](#getdatasource)
    * [cancel](#cancel)
    * [cleanup](#cleanup)
    * [loadData](#loaddata)
    * [The full DataFetcher](#the-full-datafetcher)
* [Finishing off the ModelLoader.](#finishing-off-the-modelloader)
* [Registering our ModelLoader with Glide.](#registering-our-modelloader-with-glide)
    * [Adding the AppGlideModule](#adding-the-appglidemodule)
    * [Picking our Registry method](#picking-our-registry-method)
    * [Implementing ModelLoaderFactory](#implementing-modelloaderfactory)
    * [Registering our ModelLoader](#registering-our-modelloader)
* [Complete sample.](#complete-sample)
* [Caveats](#caveats)
* [Advanced use cases](#advanced-use-cases)
    * [Delegating to another ModelLoader](#delegating-to-another-modelloader)
    * [Handling custom sizes in ModelLoaders](#handling-custom-sizes-in-modelloaders)
    * [BaseGlideUrlLoader](#baseglideurlloader)
        * [getUrl](#geturl)
        * [getHeaders](#getheaders)
* [Credits](#credits)

Although Glide provides out of the box support for most common types of models (URLs, Uris, file paths etc), you may occasionally run into a type that Glide doesn’t support. You may also run in to cases where you want to customize or tweak Glide’s default behavior. You may even want to integrate a new way of fetching images or a new networking library beyond those available in Glide’s [integration libraries](https://bumptech.github.io/glide/int/about.html).


Fortunately Glide is extensible. To add support for a new type of model, you’ll need to follow three steps:

1. Implement a <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>
2. Implement a <code>[DataFetcher](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.html)</code> that can be returned by your <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>
3. Register your new <code>[ModelLoader](https://bumptech.github.io/glide/int/about.html)</code> with Glide using an <code>[AppGlideModule](https://bumptech.github.io/glide/doc/configuration.html#applications)</code> (or <code>[LibraryGlideModule](https://bumptech.github.io/glide/doc/configuration.html#libraries)</code> if you’re working on a library rather than an application).

So that we have something to follow along with, let’s implement a custom <code>ModelLoader</code> that takes Base64 encoded image Strings and decodes them with Glide. Note that if you actually want to do this in your application, it would be better to retrieve the Base64 encoded Strings in your <code>ModelLoader</code> so that you can avoid the CPU and memory overhead of loading them into memory if Glide has previously cached your image.


For our purposes though, loading a Base64 image should provide a simple example, even if it might be a bit inefficient in the real world.


## **Writing the ModelLoader.**

The first step is to implement the <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code> interface. Before we do so, we need to make two decisions:

1. What type of Model should we handle?
2. What type of Data should we produce for that Model?

In this case we’d like to handle base 64 encoded Strings, so that means `String` is probably a reasonable choice for our Model type. Later on we’ll need something more specific than just any random String, but for now, String is sufficient to start our implementation.


Next we need to decide what type of data we should try to obtain from our String. By default, Glide provides image decoders for two types of data:

1. <code>[InputStream](https://developer.android.com/reference/java/io/InputStream.html)</code>
2. <code>[ByteBuffer](https://developer.android.com/reference/java/nio/ByteBuffer.html)</code>

    Glide also provides default support for <code>[ParcelFileDescriptor](https://developer.android.com/reference/android/os/ParcelFileDescriptor.html)</code> for decoding videos.


    Since we’re decoding an Image, we probably want <code>[InputStream](https://developer.android.com/reference/java/io/InputStream.html)</code> or <code>[ByteBuffer](https://developer.android.com/reference/java/nio/ByteBuffer.html)</code>. Since we already have all of the data in memory and the methods in <code>[Base64](https://developer.android.com/reference/android/util/Base64.html)</code>, which we’ll be using to do the actual decoding, return <code>byte[]</code>, <code>[ByteBuffer](https://developer.android.com/reference/java/nio/ByteBuffer.html)</code> is probably the best choice for our data.


### **An empty implementation**

Now that we know our `Model` and `Data` types, we can create a class that accepts the right types and returns default values:



```
package judds.github.com.base64modelloaderexample;

import android.support.annotation.Nullable;
import com.bumptech.glide.load.Options;
import com.bumptech.glide.load.model.ModelLoader;
import java.io.InputStream;
import java.nio.ByteBuffer;

/**
 * Loads an {@link InputStream} from a Base 64 encoded String.
 */
public final class Base64ModelLoader implements ModelLoader<String, ByteBuffer> {

  @Nullable
  @Override
  public LoadData<ByteBuffer> buildLoadData(String model, int width, int height, Options options) {
    return null;
  }

  @Override
  public boolean handles(String model) {
    return false;
  }
}
```



Of course this `ModelLoader` won’t do much, but it’s a start.


### **Implementing <code>handles()</code></strong>**

The next step is to implement the <code>[handles()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#handles-Model-)</code> method. As we mentioned earlier, there are a number of different types of Models that a String might be, including:



1. URLs
2. Uris
3. File paths

The `handles()` method allows the `ModelLoader` to efficiently check each model and avoid loading unsupported types.


To make our jobs easier, let’s assume that our base64 encoded Strings will actually be passed to us as [data URIs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs). The goal of the handles method is to identify any Strings passed in to Glide that match the data uri format and return `true` for only those strings.


Fortunately the data URI format seems pretty straight forward, so we can just check for the `data:` prefix:



```
 @Override
  public boolean handles(String model) {
    return model.startsWith("data:");
  }
```



Depending on how robust we want our implementation to be, we might also want to check for the embedded image type or the format of the data, so that we don’t try to load the bytes for an html page for example. We’ll skip that here for now for simplicities sake.


### **Implementing <code>buildLoadData</code></strong>**


Now that we’re able to identify data URIs, the next step is to provide an object that can decode the actual `ByteBuffer` if it’s not already in cache for our given model, dimensions, and options. To do so we need to implement the <code>[buildLoadData](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#buildLoadData-Model-int-int-com.bumptech.glide.load.Options-)</code> method.

To start, our method just returns `null`, which is perfectly valid, although not very useful:


```
 @Nullable
  @Override
  public LoadData<ByteBuffer> buildLoadData(String model, int width, int height, Options options) {
    return null;
  }
```

To make our method more useful, let’s start by returning a new <code>[LoadData&lt;ByteBuffer>](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.LoadData.html)</code> object. To do so, we’re going to need two things:



1. A <code>[Key](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/Key.html)</code> that will be used as part of our disk cache keys (the model’s <code>equals()</code> and <code>hashCode()</code> methods are used for the in memory cache key).
2. A <code>[DataFetcher](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.html)</code> that can obtain a <code>ByteBuffer</code> for our particular model.

#### <strong>Picking the <code>Key</code></strong>


The <code>[Key](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/Key.html)</code> for the disk cache is straight forward in this case because our model type is a <code>String</code>. If you have a model type that can be serialized using <code>[toString()](https://developer.android.com/reference/java/lang/Object.html#toString())</code>, you can just pass your model into a new <code>[ObjectKey](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/signature/ObjectKey.html)</code>:



```
Key diskCacheKey = new ObjectKey(model);
```


Otherwise, you’d want to implement the <code>[Key](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/Key.html)</code> interface here too, making sure that the <code>equals()</code>, <code>hashCode()</code> and <code>[updateDiskCacheKey](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/Key.html#updateDiskCacheKey-java.security.MessageDigest-)</code> methods were all filled out and uniquely and consistently identified your particular model.


Since we’re literally working with Strings here, <code>[ObjectKey](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/signature/ObjectKey.html)</code> will work just fine.


#### **Picking the DataFetcher**

Since we’re adding support for a new Model, we’re actually going to want a custom DataFetcher. In some cases <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>s may actually just do a bit of parsing in <code>[buildLoadData](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#buildLoadData-Model-int-int-com.bumptech.glide.load.Options-)</code> and delegate to another <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>, but we’re not so lucky here.

For now, let’s just pass in `null` here, even though it’s not a valid value, and move on to our actual `DataFetcher` implementation:


```
 @Nullable
  @Override
  public LoadData<ByteBuffer> buildLoadData(String model, int width, int height, Options options) {
    return new LoadData<>(new ObjectKey(model), /*fetcher=*/ null);
  }
```



## **Writing the <code>DataFetcher</code></strong>**

Like <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>, the <code>[DataFetcher](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.html)</code> interface is generic and requires us to specify the type of data we expect it to return. Fortunately we already decided that we wanted to support loading <code>ByteBuffer</code>s, so there’s no difficult decision to make.


As a result, we can quickly stub out an implementation:


```
public class Base64DataFetcher implements DataFetcher<ByteBuffer> {

  @Override
  public void loadData(Priority priority, DataCallback<? super ByteBuffer> callback) {}

  @Override
  public void cleanup() {}

  @Override
  public void cancel() {}

  @NonNull
  @Override
  public Class<ByteBuffer> getDataClass() {
    return null;
  }

  @NonNull
  @Override
  public DataSource getDataSource() {
    return null;
  }
}
```



Although there are a number of methods here, most of them are actually pretty easy to implement.


#### **getDataClass**


    `getDataClass()` is trivial, we’re loading `ByteBuffer`:


```
 @NonNull
  @Override
  public Class<ByteBuffer> getDataClass() {
    return ByteBuffer.class;
  }
```



#### **getDataSource**


`getDataSource()` is almost as trivial, but it has some implications. Glide’s default caching strategy is different for local images than it is for remote images. Glide assumes that it’s easy and cheap to retrieve local images, so we default to caching images after they’ve been downsampled and transformed. In contrast, Glide assumes that it’s difficult and expensive to retrieve remote images, so we default to caching the original data we retrieved.


For base64 `String`s, the best choice for your app might depend on how you retrieve the `String`s. If they’re loaded from a local database, `DataSource.LOCAL` makes the most sense. If you’re retrieving them via HTTP every time, `DataSource.REMOTE` is a better choice.


Let’s assume the `String`s are obtained locally for now:


```
 @NonNull
  @Override
  public DataSource getDataSource() {
    return DataSource.LOCAL;
  }
```



#### **cancel**

For networking libraries or long running loads where cancellation is possible, it’s a good idea to implement the `cancel()` method. Doing so will help speed up other queued loads and will save some amount of CPU, memory, or other resources.


In our case, <code>[Base64](https://developer.android.com/reference/android/util/Base64.html)</code> doesn’t offer a cancellation API, so we can leave it blank:


```
 @Override
  public void cancel() {
    // Intentionally empty.
  }
```



####  **cleanup**


`cleanup()` is an interesting one. If you’re loading an <code>[InputStream](https://developer.android.com/reference/java/io/InputStream.html)</code> or opening any kind of I/O resources, you absolutely must close and clean up the <code>InputStream</code> or resource in the <code>cleanup()</code> method.


However, in our case we’re just decoding an in memory model into in memory data. As a result, there’s nothing to clean up, so our method can also be empty:


```
 @Override
  public void cleanup() {
    // Intentionally empty only because we're not opening an InputStream or another I/O resource!
  }
```


Danger! Make sure that if you open an I/O resource or an `InputStream` you must close it here! We can only get away with leaving this method blank because we’re not doing so! :)


#### **loadData**

Now for the fun part! `loadData()` is the method where Glide expects you to do your heavy lifting. You can queue an asynchronous task, start a network request, load some data from disk or whatever you’d like. `loadData()` is always called on one of Glide’s background threads. A given `DataFetcher` will only be used on a single background thread at a time, so it doesn’t need to be thread safe. However, multiple `DataFetcher`s may be run in parallel, so any shared resources accessed by `DataFetcher`s should be thread safe.


 `loadData()` provides two arguments:



1. <code>[Priority](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/Priority.html)</code>, which can be used to prioritized requests if you’re using a networking library or other queueing system.
2. <code>[DataCallback](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.DataCallback.html)</code> which should be called with either your decoded data, or an error message if your load fails for any reason.

    We can either queue an async task and call the given <code>[DataCallback](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.DataCallback.html)</code> asynchronously, or we can do some work inside the <code>loadData()</code> method and call the <code>[DataCallback](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.DataCallback.html)</code> directly.


In our case we don’t have any network or other queue to call, so we can just do our work in line.


Note that one important thing is missing here. We don’t have a reference to our model! This is because each <code>[DataFetcher](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.html)</code> is basically a closure that can be used to obtain data for a particular model. As a result, Glide expects you to pass the model into the constructor of the <code>[DataFetcher](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/data/DataFetcher.html)</code>:



```
 private final String model;

  Base64DataFetcher(String model) {
    this.model = model;
  }
```



As it turns out, our `loadData()` method is now actually rather simple. We just need to parse out the base64 section of the data uri:


```
 private String getBase64SectionOfModel() {
    // See https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs.
    int startOfBase64Section = model.indexOf(',');
    return model.substring(startOfBase64Section + 1);
  }
```


Then we need to decode the `byte[]`s of the base64 section:


```
 byte[] data = Base64.decode(base64Section, Base64.DEFAULT);
```


And convert it to a `ByteBuffer`:


```
 ByteBuffer byteBuffer = ByteBuffer.wrap(data);
```



Then we just need to call the callback with our decoded `ByteBuffer`:


```
 callback.onDataReady(byteBuffer);
```


With everything together, here’s the complete `loadData()` implementation:


```
 @Override
  public void loadData(Priority priority, DataCallback<? super ByteBuffer> callback) {
    String base64Section = getBase64SectionOfModel();
    byte[] data = Base64.decode(base64Section, Base64.DEFAULT);
    ByteBuffer byteBuffer = ByteBuffer.wrap(data);
    callback.onDataReady(byteBuffer);
  }

  private String getBase64SectionOfModel() {
    // See https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs.
    int startOfBase64Section = model.indexOf(',');
    return model.substring(startOfBase64Section + 1);
  }
```



#### **The full DataFetcher**

 Now that we’ve got all the methods in `DataFetcher` implemented, let’s take one more look at it all together:


```
package judds.github.com.base64modelloaderexample;

import android.support.annotation.NonNull;
import android.util.Base64;
import com.bumptech.glide.Priority;
import com.bumptech.glide.load.DataSource;
import com.bumptech.glide.load.data.DataFetcher;
import java.nio.ByteBuffer;

public class Base64DataFetcher implements DataFetcher<ByteBuffer> {

  private final String model;

  Base64DataFetcher(String model) {
    this.model = model;
  }

  @Override
  public void loadData(Priority priority, DataCallback<? super ByteBuffer> callback) {
    String base64Section = getBase64SectionOfModel();
    byte[] data = Base64.decode(base64Section, Base64.DEFAULT);
    ByteBuffer byteBuffer = ByteBuffer.wrap(data);
    callback.onDataReady(byteBuffer);
  }

  private String getBase64SectionOfModel() {
    // See https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs.
    int startOfBase64Section = model.indexOf(',');
    return model.substring(startOfBase64Section + 1);
  }

  @Override
  public void cleanup() {
    // Intentionally empty only because we're not opening an InputStream or another I/O resource!
  }

  @Override
  public void cancel() {
    // Intentionally empty.
  }

  @NonNull
  @Override
  public Class<ByteBuffer> getDataClass() {
    return ByteBuffer.class;
  }

  @NonNull
  @Override
  public DataSource getDataSource() {
    return DataSource.LOCAL;
  }
}
```



## **Finishing off the ModelLoader.**


 Back when we were working on our <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>, we left the <code>[buildLoadData](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#buildLoadData-Model-int-int-com.bumptech.glide.load.Options-)</code> method a little incomplete and returned <code>null</code> instead of a valid <code>DataFetcher</code>:


```
 @Nullable
  @Override
  public LoadData<ByteBuffer> buildLoadData(String model, int width, int height, Options options) {
    return new LoadData<>(new ObjectKey(model), /*fetcher=*/ null);
  }
```


Now that we have a `DataFetcher` implementation, we can fill that part in:


```
 @Override
  public LoadData<ByteBuffer> buildLoadData(String model, int width, int height, Options options) {
    return new LoadData<>(new ObjectKey(model), new Base64DataFetcher(model));
  }
```


We can drop `@Nullable` since we’re never actually going to return `null` in our implementation. If we were delegating to a wrapped `ModelLoader`, we’d want to check that `ModelLoader`s return value and be sure to return `null` if it returned `null`. In some cases we may actually discover while attempting to parse our data that we can’t actually load it, in which case we can also return `null`.


## **Registering our ModelLoader with Glide.**

We’re almost done, but there’s one last step. Our `ModelLoader` implementation is complete, but totally unused. To finish off our project, we need to tell Glide about our `ModelLoader` so that Glide knows to use it.


###  **Adding the AppGlideModule**

To do so, we’re going to follow the steps on [the configuration page](https://bumptech.github.io/glide/doc/configuration.html#applications) for our application and add an <code>[AppGlideModule](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/module/AppGlideModule.html)</code> if you haven’t already done so:


```
package judds.github.com.base64modelloaderexample;

import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public class MyAppGlideModule extends AppGlideModule { }
```

Don’t forget to add a dependency on Glide’s annotation processor to your build.gradle file as well:


```
annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
```
Next we want to get at Glide’s <code>[Registry](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/Registry.html)</code>, so we’ll implement the <code>[registerComponents](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/module/LibraryGlideModule.html#registerComponents-android.content.Context-com.bumptech.glide.Glide-com.bumptech.glide.Registry-)</code> method in our <code>AppGlideModule</code>:


```
package judds.github.com.base64modelloaderexample;

import com.bumptech.glide.annotation.GlideModule;
import com.bumptech.glide.module.AppGlideModule;

@GlideModule
public class MyAppGlideModule extends AppGlideModule { 
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    // TODO: implement this.
  }
}
```



###  **Picking our Registry method**


To tell Glide about our `ModelLoader`, we need to add it to the <code>[Registry](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/Registry.html)</code> using one of the available methods for <code>ModelLoader</code>s.


 `ModelLoaders` are stored in a list in the order they are registered. When you start a new load, Glide looks at all the registered `ModelLoader`s for the model type you provide in the order they were registered and attempts them in order.


As a result, if you’re adding support for a new Model type you typically want to <code>[prepend()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/Registry.html#prepend-java.lang.Class-java.lang.Class-com.bumptech.glide.load.model.ModelLoaderFactory-)</code> your <code>ModelLoader</code> so that Glide attempts it before the default <code>ModelLoader</code>s. In our case, we’re doing exactly that, adding support for a new type of model, so we want <code>[prepend()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/Registry.html#prepend-java.lang.Class-java.lang.Class-com.bumptech.glide.load.model.ModelLoaderFactory-)</code>.


 However, that there’s one more wrinkle here. <code>[prepend()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/Registry.html#prepend-java.lang.Class-java.lang.Class-com.bumptech.glide.load.model.ModelLoaderFactory-)</code> takes a <code>[ModelLoaderFactory](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoaderFactory.html)</code>, not a <code>[ModelLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html)</code>. Doing so allows you to delegate to other <code>ModelLoader</code>s, even when they’re registered dynamically, but it also adds an interface you have to implement when defining new loaders.


### **Implementing ModelLoaderFactory**


Fortunately the <code>[ModelLoaderFactory](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoaderFactory.html)</code> interface is quite simple, so we can add it easily:


```
package judds.github.com.base64modelloaderexample;

import com.bumptech.glide.load.model.ModelLoader;
import com.bumptech.glide.load.model.ModelLoaderFactory;
import com.bumptech.glide.load.model.MultiModelLoaderFactory;
import java.nio.ByteBuffer;

public class Base64ModelLoaderFactory implements ModelLoaderFactory<String, ByteBuffer> {

  @Override
  public ModelLoader<String, ByteBuffer> build(MultiModelLoaderFactory unused) {
    return new Base64ModelLoader();
  }

  @Override
  public void teardown() { 
    // Do nothing.
  }
}
```



The types for <code>[ModelLoaderFactory](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoaderFactory.html)</code> match those used in our <code>ModelLoader</code> exactly.


### **Registering our ModelLoader**


 Finally we just need to update our `AppGlideModule` to use our new Factory:


```
@GlideModule
public class MyAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.prepend(String.class, ByteBuffer.class, new Base64ModelLoaderFactory());
  }
}
```

And that’s it!

Now we can just take any data uri with a base64 encoded image and load it with Glide and it just works:


```
String dataUri = "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAYA..."
Glide.with(fragment)
  .load(dataUri)
  .into(imageView);
```



## **Complete sample.**

A complete sample project using the code we’ve written is available here: [https://github.com/sjudd/Base64ModelLoaderExample](https://github.com/sjudd/Base64ModelLoaderExample/commit/ae004dc4b325ee39814f197cc196d7371fbccdf1).

The commits are in the same order we wrote the code in the sample:



1. [An empty project with a blank Activity](https://github.com/sjudd/Base64ModelLoaderExample/commit/d9ee7eb9285ed1a7279cc085b3abd0f1369f92dd)
2. [A ModelLoader that just implements handles](https://github.com/sjudd/Base64ModelLoaderExample/commit/83ae04155b79056487299f65f70e172c11ff53ae)
3. [A data fetcher implementation](https://github.com/sjudd/Base64ModelLoaderExample/commit/70a4facda7504c375ca9150ea2a6789077bbd7e1)
4. [A ModelLoader with a complete buildLoadData implementation](https://github.com/sjudd/Base64ModelLoaderExample/commit/8c641cb1be3afbf1ff0d8bcba7b37b1778f06dc4)
5. [An AppGlideModule and registered ModelLoader](https://github.com/sjudd/Base64ModelLoaderExample/commit/d4e1cd9dcc011bb6d2910301c5783290fbe3bb89)
6. [An example data Uri loaded into a View](https://github.com/sjudd/Base64ModelLoaderExample/commit/ae004dc4b325ee39814f197cc196d7371fbccdf1)

## **Caveats**
 As it turns out, Glide already supports Data URIs, so no need to do anything if you want load base64 strings as data URIs. This code is for example purposes only.

If you did want to implement support for data Uris, you’d probably want to do some more error checking on `handles()` or in the `DataFetcher` to handle truncated strings where your indexes might exceed the bounds of the uri.


## **Advanced use cases**


There are a couple of more advanced use cases that don’t fit into our tutorial above. We’ll address them here individually.


### **Delegating to another ModelLoader**


One thing we mentioned earlier, but didn’t discuss in detail is that Glide allows you to delegate to an existing ModelLoader in a custom ModelLoader. It’s not uncommon to have a custom model type that Glide doesn’t understand, but be able to relatively easily extract a model type that Glide does understand, like a URL, Uri, or file path, from the custom type.

With delegation, you can add support for your custom model by extracting the model that Glide does understand and delegating.


For example, in Glide’s [Giphy sample app](https://bumptech.github.io/glide/ref/samples.html#giphy), we obtain a JSON object from Giphy’s API that contains a set of URLs:



```
/**
 * A POJO mirroring an individual GIF image returned from Giphy's API.
 */
public static final class GifResult {
  public String id;
  GifUrlSet images;

  @Override
  public String toString() {
    return "GifResult{" + "id='" + id + '\'' + ", images=" + images
        + '}';
  }
}
```



Although we could extract the urls in our View logic and do something like this:


```
Glide.with(fragment)
  .load(gifResult.images.fixed_width)
  .into(imageView);
```



It’s cleaner if we can just pass in the `GifResult` directly:


```
Glide.with(fragment)
  .load(gifResult)
  .into(imageView);
```



If we had to re-write all of our URL handling logic to do that, it wouldn’t be worth the effort. If we can delegate though, we end up with a fairly simple `ModelLoader` implementation:


```
public final class GiphyModelLoader extends BaseGlideUrlLoader<Api.GifResult> {
  private final ModelLoader<GlideUrl, InputStream> urlLoader;

  private GiphyModelLoader(ModelLoader<GlideUrl, InputStream> urlLoader) {
    this.urlLoader = urlLoader;
  }

  @Override
  public boolean handles(@NonNull Api.GifResult model) {
    return true;
  }

  @Override
  public LoadData<InputStream> buildLoadData(
      @NonNull Api.GifResult model, int width, int height, @NonNull Options options) {
    return urlLoader.buildLoadData(model.images.fixed_width, width, height, options);
  }
}
```



The `ModelLoader&lt;GlideUrl, InputStream>` required by our `ModelLoader`’s constructor is provided by our `ModelLoaderFactory` which can look up the currently registered `ModelLoader`s for a given model and data type (`GlideUrl` and `InputStream` in this case):


```
/**
 * The default factory for {@link com.bumptech.glide.samples.giphy.GiphyModelLoader}s.
 */
public static final class Factory implements ModelLoaderFactory<GifResult, InputStream> {
  @Override
  public ModelLoader<Api.GifResult, InputStream> build(MultiModelLoaderFactory multiFactory) {
    return new GiphyModelLoader(multiFactory.build(GlideUrl.class, InputStream.class));
  }

  @Override public void teardown() {}
}
```



Glide’s `ModelLoader`s are built lazily and torn down if new `ModelLoader`s are registered so that you never end up using a stale `ModelLoader` when you use this delegation pattern. As a result, our `GiphyModelLoader` is totally decoupled from the networking library we actually use to load the url.


The <code>[MultiModelLoaderFactory](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/MultiModelLoaderFactory.html)</code> can be used to obtain any registered <code>ModelLoader</code>. If multiple <code>ModelLoader</code>s are registered for a given type, the <code>MultiModelLoaderFactory</code> will return a wrapping <code>ModelLoader</code> that will attempt each <code>ModelLoader</code> that returns <code>true</code> from <code>[handles()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#handles-Model-)</code> for a given model in order until one succeeds.


### **Handling custom sizes in ModelLoaders**


Even with delegation, the Giphy example above might seem like a fair amount of work for a slightly nicer API. However, there are additional benefits to having your own `ModelLoader`, especially for APIs like Giphy’s where you have multiple URLs you can choose from.


Although we implemented <code>[buildLoadData()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#buildLoadData-Model-int-int-com.bumptech.glide.load.Options-)</code> previously for our base 64 <code>ModelLoader</code>, we never discussed the arguments provided other than the model. <code>buildLoadData()</code> also passes in a width and a height that can be used to select the most appropriately sized image, which can save bandwidth, memory, CPU, disk space etc by only retrieving, caching, and decoding the smallest image necessary.


The width and height passed in to `buildLoadData()` are either those provided by the <code>[Target](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/request/target/Target.html)</code> or, if specified, the <code>[override()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/request/RequestOptions.html#override-int-int-)</code> dimensions for the request. If you’re loading into an <code>ImageView</code> the width and height provided to <code>buildLoadData()</code> are the width and height of the <code>ImageView</code> (again unless <code>override()</code> is used). If you use <code>Target.SIZE_ORIGINAL</code>, the width and height will be the constant <code>Target.SIZE_ORIGINAL</code>.


The actual <code>[GiphyModelLoader](https://github.com/bumptech/glide/blob/b4b45791cca6b72345a540dcaa71a358f5706276/samples/giphy/src/main/java/com/bumptech/glide/samples/giphy/GiphyModelLoader.java#L31)</code> has a simple example of using the dimensions provided to <code>buildLoadData()</code> to pick the best available url:


```
@Override
protected String getUrl(Api.GifResult model, int width, int height, Options options) {
  Api.GifImage fixedHeight = model.images.fixed_height;
  int fixedHeightDifference = getDifference(fixedHeight, width, height);
  Api.GifImage fixedWidth = model.images.fixed_width;
  int fixedWidthDifference = getDifference(fixedWidth, width, height);
  if (fixedHeightDifference < fixedWidthDifference && !TextUtils.isEmpty(fixedHeight.url)) {
    return fixedHeight.url;
  } else if (!TextUtils.isEmpty(fixedWidth.url)) {
    return fixedWidth.url;
  } else if (!TextUtils.isEmpty(model.images.original.url)) {
    return model.images.original.url;
  } else {
    return null;
  }
}
```



In Glide’s [Flickr sample app](https://bumptech.github.io/glide/ref/samples.html#flickr), we see a similar pattern, although somewhat more robust because of the large variety of available thumbnail sizes.


 If you have access to an API that either allows you to specify a specific size to request or that offers a variety of thumbnail sizes, using a custom `ModelLoader` can significantly improve the performance of your application.


### **BaseGlideUrlLoader**


To save some of the boiler plate required to write a custom `ModelLoader` that just delegates to the default networking library, Glide includes the <code>[BaseGlideUrlLoader](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/stream/BaseGlideUrlLoader.html)</code> abstract class. A couple of our previous examples, including the <code>[GiphyModelLoader](https://github.com/bumptech/glide/blob/b4b45791cca6b72345a540dcaa71a358f5706276/samples/giphy/src/main/java/com/bumptech/glide/samples/giphy/GiphyModelLoader.java#L31)</code> and the <code>[FlickrModelLoader](https://github.com/bumptech/glide/blob/b4b45791cca6b72345a540dcaa71a358f5706276/samples/flickr/src/main/java/com/bumptech/glide/samples/flickr/FlickrModelLoader.java#L21)</code> make use of this class.

`BaseGlideUrlLoader` provides some basic caching to minimize `String` allocations and two convenience methods:



1. <code>[getUrl()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/stream/BaseGlideUrlLoader.html#getUrl-Model-int-int-com.bumptech.glide.load.Options-)</code> which returns a <code>String</code> url for a given model
2. <code>[getHeaders()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/stream/BaseGlideUrlLoader.html#getHeaders-Model-int-int-com.bumptech.glide.load.Options-)</code> which can be optionally implemented to returns a set of HTTP <code>[Headers](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/Headers.html)</code> for a given model and dimensions if you need to add an authentication or other type of header.

#### <strong>getUrl</strong>


If you read the earlier section about handling custom sizes, you might have noticed that the method in <code>[GiphyModelLoader](https://github.com/bumptech/glide/blob/b4b45791cca6b72345a540dcaa71a358f5706276/samples/giphy/src/main/java/com/bumptech/glide/samples/giphy/GiphyModelLoader.java#L31)</code> we referenced isn’t actually <code>[buildLoadData()](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/ModelLoader.html#buildLoadData-Model-int-int-com.bumptech.glide.load.Options-)</code>. It’s actually just the <code>getUrl()</code> convenience method:



```
@Override
protected String getUrl(Api.GifResult model, int width, int height, Options options) {
  Api.GifImage fixedHeight = model.images.fixed_height;
  int fixedHeightDifference = getDifference(fixedHeight, width, height);
  Api.GifImage fixedWidth = model.images.fixed_width;
  int fixedWidthDifference = getDifference(fixedWidth, width, height);
  if (fixedHeightDifference < fixedWidthDifference && !TextUtils.isEmpty(fixedHeight.url)) {
    return fixedHeight.url;
  } else if (!TextUtils.isEmpty(fixedWidth.url)) {
    return fixedWidth.url;
  } else if (!TextUtils.isEmpty(model.images.original.url)) {
    return model.images.original.url;
  } else {
    return null;
  }
}
```



Using `BaseGlideUrlLoader` allows you to skip constructing the disk cache key and `LoadData` and allows you to avoid dealing with delegation, aside from the `ModelLoader&lt;GlideUrl, InputStream>` you have to pass in to the constructor.


#### **getHeaders**


Although Glide’s sample apps don’t need to use `getHeaders()`, it’s not uncommon to have to attach some form of authentication when retrieving non-public images. The `getHeaders()` method can be optionally implemented to return any set of HTTP headers that is appropriate for a given model.


For example, if you had a string authorization token, you might use the <code>[LazyHeaders](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/LazyHeaders.html)</code> class to write something like this:


```
@Nullable
@Override
protected Headers getHeaders(GifResult gifResult, int width, int height, Options options) {
  return new LazyHeaders.Builder()
      .addHeader("Authorization", getAuthToken())
      .build();
}
```



 If your `getAuthToken()` method is especially expensive, you should use the <code>[LazyHeaderFactory](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/model/LazyHeaderFactory.html)</code> instead:


```
@Override
protected Headers getHeaders(GifResult gifResult, int width, int height, Options options) {
  return new LazyHeaders.Builder()
      .addHeader("Authorization", new LazyHeaderFactory() {
        @Nullable
        @Override
        public String buildHeader() {
          return getAuthToken();
        }
      })
      .build();
}
```


Using `LazyHeaderFactory` will avoid running expensive calls until the HTTP request is made in the `DataFetcher`. Although the `ModelLoader` methods are called on background threads, `buildLoadData()` is called, even if the corresponding image is in Glide’s disk cache. As a result, it’s wasteful to perform expensive work during the `buildLoadData()` method or any of the `BaseGlideUrlLoader` methods because the result may noto be used. Using `LazyHeaderFactory` will defer the work, saving a significant amount of time for expensive to acquire headers.


## **Credits**


Thanks to jasonch@ for Glide’s [data uri ModelLoader implementation](https://github.com/bumptech/glide/blob/c3dafde00a061bafcd43a739336ca3503af13a7d/library/src/main/java/com/bumptech/glide/load/model/DataUrlLoader.java#L19).
