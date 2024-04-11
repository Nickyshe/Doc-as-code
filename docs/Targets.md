

## **Targets **



* [About](#about)
* [Specifying Targets](#specifying-targets)
* [Cancellation and re-use](#cancellation-and-re-use)
    * [Clear](#clear)
* [Sizes and dimensions](#sizes-and-dimensions)
    * [View Targets](#view-targets)
        * [Performant View Sizes](#performant-view-sizes)
        * [Alternatives](#alternatives)
    * [Custom Targets](#custom-targets)
* [Animated Resources and custom Targets.](#animated-resources-and-custom-targets)


### **About**

<code>[Targets](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/Target.html)</code> in Glide act as mediators between requests and requestors. Targets are responsible for displaying placeholders, loaded resources, and determining the appropriate dimensions for each request. The most frequently used Targets are <code>[ImageViewTargets](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/ImageViewTarget.html)</code> that display placeholders, Drawables, and Bitmaps using ImageView. Users can also implement their own Targets, or subclass any of the available base classes.


### **Specifying Targets**

The <code>[into(Target)](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-Y-)</code> method is used not only to start each request, but also to specify the Target that will receive the results of the request:


```
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
```


Glide provides a helper method for <code>[into(ImageView)](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-android.widget.ImageView-)</code> that takes an <code>ImageView</code> and wraps it in a <code>[ImageViewTarget](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/ImageViewTarget.html)</code> appropriate for the requested resource type for you:


```
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(imageView);
```



### **Cancellation and re-use**

Note that both <code>[into(Target)](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-Y-)</code> and <code>[into(ImageView)](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#into-android.widget.ImageView-)</code> return a <code>Target</code> instance. If you re-use this <code>Target</code> to start a new load in the future, any previously started requests will be cancelled and their resources released:


```
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
... 
// Some time in the future:
Glide.with(fragment)
  .load(newUrl)
  .into(target);
```


You can also use the returned `Target` to <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> out any pending loads and release any associated resources without starting a new load:


```
Target<Drawable> target = 
  Glide.with(fragment)
    .load(url)
    .into(new Target<Drawable>() {
      ...
    });
... 
// Some time in the future:
Glide.with(fragment).clear(target);
```


Glide’s <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code> subclasses store information about each request in the <code>View</code> using the Android Framework’s <code>[getTag()](https://developer.android.com/reference/android/view/View.html#getTag())</code> and <code>[setTag()](https://developer.android.com/reference/android/view/View.html#setTag(java.lang.Object))</code> methods, so if you’re using a <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code> subclass or loading into an <code>ImageView</code>, you can re-use or clear the <code>View</code> directly:


```
Glide.with(fragment)
  .load(url)
  .into(imageView);

// Some time in the future:
Glide.with(fragment).clear(imageView);

// Or:
Glide.with(fragment)
  .load(newUrl)
  .into(imageView);
```


In addition, **for <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code>s only</strong>, you can pass in a new instance to each load or clear call and allow Glide to retrieve information about previous loads from the <code>View</code>s tags:


```
Glide.with(fragment)
  .load(url)
  .into(new DrawableImageViewTarget(imageView));

// Some time in the future:
Glide.with(fragment)
  .load(newUrl)
  .into(new DrawableImageViewTarget(imageView));
```


This will **not** work unless your `Target` extends <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code> or implements <code>[setRequest()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#setRequest-com.bumptech.glide.request.Request-)</code> and <code>[getRequest()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#getRequest--)</code> in a way that allows you to retrieve previous requests in new <code>Target instances</code>.


#### **Clear**

It’s always good practice to <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> out any <code>Target</code> you create when you’re done with any resource (<code>Bitmap</code>, <code>Drawable</code> etc) that you’ve loaded into it. You should use <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> even if you think your request has finished to let Glide re-use any resources (Bitmaps in particular) used by the load. Failing to <code>[clear()](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> a <code>Target</code> can waste CPU and memory, block more important loads, or even result in the wrong image appearing if you have two <code>Target</code>s that display images in the same surface (View, Notification, RPC etc). Clear is especially critical for <code>Target</code>s like <code>[SimpleTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html)</code> that can’t keep track of previous requests from other <code>[SimpleTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html)</code> instances.


### **Sizes and dimensions**

By default, Glide uses the size provided by Targets via the <code>[getSize()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-)</code> method as the target size for the request. Doing so allows Glide to choose an appropriate url, downsample, crop, and Transform size appropriate images to minimize memory usage, and make sure loads are as fast as possible.


#### **View Targets**

<code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code>s implement <code>[getSize()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-)</code> by inspecting the attributes of the View and/or using an <code>[OnPreDrawListener](https://developer.android.com/reference/android/view/ViewTreeObserver.OnPreDrawListener.html)</code> to measure the View immediately before it is rendered. As a result, Glide can automatically resize most images to match the <code>View</code> they will be displayed in. Loading smaller images lets Glide load them faster (after they’ve been disk cached), use less memory, and increases the hit rate of Glide’s BitmapPool if the View sizes are consistent.

The logic `ViewTarget` uses is as follows:



1. If the `View`’s layout parameter dimensions are > 0 and > padding, use the layout parameters
2. If the `View` dimensions are > 0 and > padding, use the view dimensions
3. If the `View` layout parameters are `wrap_content` and at least one layout pass has occurred, log a warning suggesting to use `Target.SIZE_ORIGINAL` or some other fixed size via `override()` and use the screen dimensions.
4. Otherwise (layout parameters are `match_parent`, `0`, or `wrap_content` and no layout pass has occurred), wait for a layout pass, then go back to step 1.

Sometimes when using `RecyclerView`, a `View` may be re-used and retain the size from a previous position that will be changed for the current position. To handle those cases, you can create a new [`ViewTarget` and pass in `true` for `waitForLayout`]:


```
@Override
public void onBindViewHolder(VH holder, int position) {
  Glide.with(fragment)
    .load(urls.get(position))
    .into(new DrawableImageViewTarget(holder.imageView, /*waitForLayout=*/ true));
```



##### **Performant View Sizes**

In general Glide provides the fastest and most predictable results when explicit dp sizes are set on Views it loads into. However when that’s not possible, Glide also provides robust support for layout weights, `match_parent` and other relative sizes using `OnPreDrawListeners`. Finally, if nothing else will work, Glide should provide reasonable behavior for `wrap_content` as well.


##### **Alternatives**

If in any case Glide seems to get View sizes wrong, you can always manually override the size, either by extending <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/ViewTarget.html)</code> and implementing your own logic, or by using the <code>[override()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#override-int-int-)</code> method in <code>RequestOptions</code>.


#### **Custom Targets**

If you’re using a custom `Target` and you’re not loading into a `View` that would allow you to subclass <code>[ViewTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/ViewTarget.html)</code>, you’ll need to implement the <code>[getSize()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-)</code> method.

The simplest possible way to implement `getSize()` is to simply call the provided callback immediately:


```
@Override
public void getSize(SizeReadyCallback cb) {
  cb.onSizeReady(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
}
```


Using `Target.SIZE_ORIGINAL` can be very inefficient or cause OOMs if your image sizes are large enough. As an alternative, You can also pass in a size to your `Target`’s constructor and provide those dimensions to the callback:


```
public class CustomTarget<T> implements Target<T> {
  private final int width;
  private final int height;


  public CustomTarget(int width, int height) {
    this.width = width;
    this.height = height;
  }

  ...

  @Override
  public void getSize(SizeReadyCallback cb) {
    cb.onSizeReady(width, height);
  }
}
```


Passing in a specific set of dimensions works well when you’re using consistent sizes in your application or when you know exactly what size you need. If you don’t know what size you need but could find out asynchronously, you can retain any callbacks you’re given in <code>[getSize()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/Target.html#getSize-com.bumptech.glide.request.target.SizeReadyCallback-)</code> in a <code>List&lt;SizeReadyCallBack></code>, run some asynchronous process and then notify the callbacks you’ve retained when you figure out what the size should be.

Be sure to also implement <code>[removeCallback](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/Target.html#removeCallback-com.bumptech.glide.request.target.SizeReadyCallback-)</code> if you retain callbacks to avoid memory leaks.

For an example see the logic in Glide’s <code>[ViewTarget](https://github.com/bumptech/glide/blob/e9cf41fbc190c9d29ce683728f52c061809c749b/library/src/main/java/com/bumptech/glide/request/target/ViewTarget.java#L89)</code>.


### **Animated Resources and custom Targets.**

If you’re just loading the `GifDrawable`, or really any resource type, into a `View` you should always use `into(ImageView)` when possible. In addition to graceful handling or new requests, most of Glide’s `ViewTarget` implementations will already take care of starting animated `Drawable`s for you. If you absolutely must have a custom `Target` be sure to either or extend `ViewTarget` or rigorously clear the `Target` returned from `into(Target)` before starting new loads or when you’re done displaying the resource you loaded.

If you’re not loading into a `View`, using `ViewTarget` directly or using a custom `Target` like <code>[SimpleTarget](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/request/target/SimpleTarget.html)</code> and you’re loading an animated resource like <code>[GifDrawable](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/resource/gif/GifDrawable.html)</code>, you need to make sure to <code>[start](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/resource/gif/GifDrawable.html#start--)</code> the animated <code>Drawable</code> in <code>onResourceReady</code>:


```
Glide.with(fragment)
  .asGif()
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(GifDrawable resource, Transition<GifDrawable> transition) {
      resource.start();
      // Set the resource wherever you need to use it.
    }
  });
```


If you’re loading either a `Bitmap` or a `GifDrawable`, you can check to see if the Drawable implements <code>[Animatable](https://developer.android.com/reference/android/graphics/drawable/Animatable.html)</code>:


```
Glide.with(fragment)
  .load(url)
  .into(new SimpleTarget<>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<GifDrawable> transition) {
      if (resource instanceof Animatable) {
        resource.start();
      }
      // Set the resource wherever you need to use it.
    }
  });
