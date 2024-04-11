

## **Resource Reuse**



* [Resources](#resources)
* [Benefits](#benefits)
    * [Dalvik](#dalvik)
* [How Glide tracks and re-uses resources](#how-glide-tracks-and-re-uses-resources)
    * [Reference counting](#reference-counting)
        * [Incrementing the reference count](#incrementing-the-reference-count)
        * [Decrementing the reference count](#decrementing-the-reference-count)
        * [Releasing resources.](#releasing-resources)
    * [Pooling](#pooling)
* [Common errors](#common-errors)
    * [Symptoms of resource re-use errors.](#symptoms-of-resource-re-use-errors)
        * [Cannot draw a recycled Bitmap](#cannot-draw-a-recycled-bitmap)
        * [Can’t call reconfigure() on a recycled bitmap](#cant-call-reconfigure-on-a-recycled-bitmap)
        * [Views flicker between images or the same image shows up in multiple views](#views-flicker-between-images-or-the-same-image-shows-up-in-multiple-views)
    * [Causes of re-use errors.](#causes-of-re-use-errors)
        * [Attempting to load two different resources into the same Target.](#attempting-to-load-two-different-resources-into-the-same-target)
        * [Loading a resource into a Target, clearing or reusing the Target, and continuing to reference the resource.](#loading-a-resource-into-a-target-clearing-or-reusing-the-target-and-continuing-to-reference-the-resource)
        * [Recycling the original Bitmap in a Transformation&lt;Bitmap>.](#recycling-the-original-bitmap-in-a-transformationbitmap)

### **Resources**

Resources in Glide include things like `Bitmaps`, `byte[]` arrays, `int[]` arrays as well as a variety of POJOs. Glide attempts to re-use resources whenever possible to limit the amount of memory churn in your application.


### **Benefits**

Excessive allocations of objects of any size can dramatically increase the garbage collection (GC) overhead of your application. Although Android’s Dalvik runtime has a much higher GC penalty than the newer ART runtime, excessive allocations will decrease the performance of your application regardless of which device your own.


#### **Dalvik**

Dalvik devices (pre Lollipop) face an exceptionally large penalty for excessive allocations that are worth discussing.

Dalvik has two basic modes for garbage collection, GC_CONCURRENT and GC_FOR_ALLOC, both of which you can see in logcat.

* GC_CONCURRENT blocks the main thread twice for about 5ms for each collection. Since each operation is less than a single frame (16ms), GC_CONCURRENT tends not to cause your application to drop frames.
* GC_FOR_ALLOC is a stop the world collection that can block the main thread for 125+ms. GC_FOR_ALLOC virtually always causes your application to drop multiple frames, resulting in visible stuttering, particularly while scrolling.

Unfortunately Dalvik seems to handle even modest allocations (a 16kb buffer for example) poorly. Repeated moderate allocations, or even a single large allocation (say for a Bitmap), will cause GC_FOR_ALLOC. Therefore, the more you allocate, the more stop the world garbage collections you incur, and the more frames your application drops.


By re-using moderate to large resources, Glide helps keep your app jank free by avoiding stop the world garbage collections as much as possible.


## **How Glide tracks and re-uses resources**

Glide takes a permissive approach to resource re-use. Glide will opportunistically re-use resources when it believes it is safe to do so, but Glide does not require calleres to recycle resources after each request. Unless a caller explicitly signals that they’re done with a resource (see below), resources will not be recycled or re-used.


### **Reference counting**

In order to determine when a resource is in use and when it is safe to be re-used, Glide keeps a reference count for each resource.


#### **Incrementing the reference count**

Each call to <code>[into()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-Y-)</code> that loads a resource increments the reference count for that resource by one. If the same resource is loaded into two different <code>[Target](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html)</code>s it will have a reference count of two after both loads complete.


#### **Decrementing the reference count**

The reference count is decremented when callers signal that they are done with the resource by:

1. Calling <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> on the <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> or <code>[Target](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html)</code> the resource was loaded in to.
2. Calling <code>[into()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-Y-)</code> on the <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> or <code>[Target](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html)</code> with a request for a new resource.

The refrence count is also decremented when the <code>RequestManager</code> associated with the request is paused or cleared. The <code>RequestManager</code> associated with the request is determined by the <code>Activity</code> or <code>Fragment</code> you pass into <code>Glide.with()</code>. The <code>RequestManager</code> can be paused or cleared manually and will also monitor the <code>Lifecycle</code> of the corresponding <code>Activity</code> or <code>Fragment</code> and automatically pause or clear requests in <code>onStop</code> and <code>onDestroy</code>. If you pass the <code>Application</code> <code>Context</code> to <code>Glide.with</code>, the <code>RequestManager</code> will not automatically pause or clear requests.


#### **Releasing resources.**


When the reference count reaches zero, the resource is released and returned to Glide for re-use. After the resource is returned to Glide for re-use it is no longer safe to continue using. As a result it’s **unsafe** to:

1. Retrieve a `Bitmap` or `Drawable` loaded into an `ImageView` using `getImageDrawable()` and display it (using `setImageDrawable()`, in an animation or `TransitionDrawable` or any other method).
2. Use <code>[SimpleTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html)</code> to load a resource into a <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> without also implementing <code>[onLoadCleared()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#onLoadCleared-android.graphics.drawable.Drawable-)</code> and removing the resource from the <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> in that callback.
3. Leave <code>[onLoadCleared()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#onLoadCleared-android.graphics.drawable.Drawable-)</code> unimplemented in <code>[CustomTarget](https://bumptech.github.io/glide/javadocs/4140/library/com.bumptech.glide.request.target/-custom-target/index.html)</code>
4. Call <code>[recycle()](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())</code> on any <code>Bitmap</code> loaded with Glide.

It’s unsafe to reference a resource after clearing the corresponding <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> or <code>[Target](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html)</code> because that resource may be destroyed or re-used to display a different image, resulting in undefined behavior, graphical corruption, or crashes in applications that continue to use those resources. For example, after being released back to Glide, <code>Bitmap</code>s may be stored in a <code>BitmapPool</code> and re-used to hold the bytes of a new image at some point in the future or they may have <code>[recycle()](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())</code> called on them (or both). In either case continuing to reference the <code>Bitmap</code> and expecting it to contain the original image is unsafe.


### **Pooling**

Although most of Glide’s recycling logic is aimed at Bitmaps, all <code>[Resource](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/Resource.html)</code> implementations can implement <code>[recycle()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/Resource.html#recycle--)</code> and pool any re-usable data they might contain. <code>[ResourceDecoder](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/ResourceDecoder.html)</code>s are free to return any implementation of the <code>[Resource](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/Resource.html)</code> API they wish, so users can customize or provide additional pooling for novel types by implementing their own <code>[Resource](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/Resource.html)</code>s and <code>[ResourceDecoder](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/ResourceDecoder.html)</code>s.


For `Bitmap`s in particular, Glide provides a <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code> interface that allows <code>[Resource](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/Resource.html)</code>s to obtain and re-use <code>Bitmap</code> objects. Glide’s <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code> can be obtained from any <code>Context</code> using the Glide singleton:



```
Glide.get(context).getBitmapPool();
```



Similarly users who want more control over `Bitmap` pooling are free to implement their own <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code>, which they can then provide to Glide using a <code>GlideModule</code>. See the [configuration page](https://bumptech.github.io/glide/doc/configuration.html#bitmap-pool) for details.


## **Common errors**

Unfortunately pooling makes it difficult to assert that a user isn’t misusing a resource or a `Bitmap`. Glide tries to add assertions where possible, but because we don’t own the underlying `Bitmap` we can’t guarantee that callers stop using `Bitmap`s or other resources when they tell us they have via <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> or a new request.


### **Symptoms of resource re-use errors.**

There are a couple of indicators that something might be going wrong with `Bitmap` or other resource pooling in Glide. A few of the most common symptoms we see are listed here, though this is not an exhaustive list.


#### **Cannot draw a recycled Bitmap**

Glide’s `BitmapPool` has a fixed size. When `Bitmap`s are evicted from the pool without being re-used, Glide will call <code>[recycle()](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())</code>. If an application inadvertently continues to hold on to the <code>Bitmap</code> even after indicating to Glide that it is safe to recycle it, the application may then attempt to draw the <code>Bitmap</code>, resulting in a crash in <code>onDraw()</code>.


This problem could be due to the fact that one target is being used for two `ImageView`s, and one of the `ImageView`s still tries to access the recycled `Bitmap` after it has been put into the `BitmapPool`. This recycling error can be hard to reproduce, due to several factors:



1. When the bitmap is put into the pool
2. When the bitmap is recycled, and
3. What the size of the `BitmapPool` and memory cache are that leads to the recycling of the `Bitmap`.

The following snippet can be put into your `GlideModule` to help making this problem easier to reproduce:



```
@Override
public void applyOptions(Context context, GlideBuilder builder) {
    int bitmapPoolSizeBytes = 1024 * 1024 * 0; // 0mb
    int memoryCacheSizeBytes = 1024 * 1024 * 0; // 0mb
    builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));
    builder.setBitmapPool(new LruBitmapPool(bitmapPoolSizeBytes));
}
```

The above code makes sure that there is no memory caching and the size of the `BitmapPool` is zero; so `Bitmap`, if happened to be not used, will be recycled right away. The problem will surface much quicker for debugging purposes.


#### **Can’t call reconfigure() on a recycled bitmap**

Resources are returned to Glide’s `BitmapPool` when they’re not in use any more. This is handled internally based on the lifecycle of a `Request` (who controls <code>[Resource](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/engine/Resource.html)</code>s). If something calls <code>[recycle()](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())</code> on those Bitmaps, but they’re still in the pool, Glide cannot re-use them and your app crashes with the above message. One key point here is that the crash will likely happen in the future at another point in your app, and not where the offending code was executed!


#### **Views flicker between images or the same image shows up in multiple views**

If a `Bitmap` is returned to the `BitmapPool` multiple times, or is returned to the pool but still held on to by a <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code>, another image may be decoded into the <code>Bitmap</code>. If this happens, the contents of the <code>Bitmap</code> are replaced with the new image. <code>View</code>s may still attempt to draw the <code>Bitmap</code> during this process, which will result either in artifacts or in the original <code>View</code> showing a new image.


### **Causes of re-use errors.**

A few common causes of re-use errors are listed below. As with symptoms, it’s difficult to be exhaustive, but these are some things you should definitely consider when trying to debug a re-use error in your application.


#### **Attempting to load two different resources into the same Target.**

There is no safe way to load multiple resources into a single Target in Glide. Users can use the <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> API to load a series of resources into a <code>[Target](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html)</code>, but it is only safe to reference each earlier resource until the next call to <code>[onResourceReady()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#onResourceReady-R-com.bumptech.glide.request.transition.Transition-)</code>.


Typically a better answer is to actually use a second <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> and load the second image into the second <code>View</code>. <code>[ViewSwitcher](https://developer.android.com/reference/android/widget/ViewSwitcher.html)</code> can work well to allow you to cross fade between two different images loaded in separate requests. You can just add the <code>ViewSwitcher</code> to your layout with two <code>ImageView</code> children and use <code>[into(ImageView)](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-android.widget.ImageView-)</code> twice, once on each child, to load the two images.

Users absolutely must load multiple resources into the same <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code> can do so by using two separate <code>[Target](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html)</code>s. To make sure that the loads don’t cancel each other, users either need to avoid the <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code> subclasses, or use a custom <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code> subclass and override <code>[setRequest()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#setRequest-com.bumptech.glide.request.Request-)</code> and <code>[getRequest()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#getRequest--)</code> so that they do not use the <code>[View](https://d.android.com/reference/android/view/View.html?is-external=true)</code>’s tag to store the <code>[Request](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/Request.html)</code>. This is advanced usage and not typically recommended.


#### **Loading a resource into a Target, clearing or reusing the Target, and continuing to reference the resource.**

The easiest way to avoid this error is to make sure that all references to a resource are nulled out when <code>[onLoadCleared()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#onLoadCleared-android.graphics.drawable.Drawable-)</code> is called. It is generally safe to load a <code>Bitmap</code> and then de-reference the <code>Target</code> and never call <code>[into()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestBuilder.html#into-Y-)</code> or <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> on the <code>Target</code> again. However, it is not safe to load a <code>Bitmap</code>, clear the <code>Target</code>, and then continue to reference the <code>Bitmap</code> later. Similarly it’s unsafe to load a resource into a <code>View</code> and then obtain the resource from the View (via <code>getImageDrawable()</code> or any other means) and continue to reference it elsewhere.


#### **Recycling the original Bitmap in a <code>Transformation&lt;Bitmap></code>.</strong>**


As the JavaDoc says in <code>[Transformation](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/Transformation.html)</code>, the original <code>Bitmap</code> passed in to <code>[transform()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html#transform-com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool-android.graphics.Bitmap-int-int-)</code> will be automatically recycled if the <code>Bitmap</code> returned by the <code>Transformation</code> is not the same instance as the one passed in to <code>[transform()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html#transform-com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool-android.graphics.Bitmap-int-int-)</code>. This is an important difference from other loader libraries, for example Picasso. <code>[BitmapTransformation](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html)</code> provides the boilerplate to handle Glide’s <code>Resource</code> creation, but the recycling is done internally, so both <code>Transformation</code> and <code>BitmapTransformation</code> must not recycle the passed-in <code>Bitmap</code> or <code>Resource</code>.

 It’s also worth noting the any intermediate `Bitmap`s that any custom `BitmapTransformation` obtains from the `BitmapPool`, but does not return from <code>[transform()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html#transform-com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool-android.graphics.Bitmap-int-int-)</code> must be either put back to the <code>BitmapPool</code> or have <code>[recycle()](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())</code>, but never both. You should never <code>[recycle()](https://developer.android.com/reference/android/graphics/Bitmap.html#recycle())</code> a <code>Bitmap</code> obtained from Glide.
