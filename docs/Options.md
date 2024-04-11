
## **Options**

* [RequestBuilder options](#requestbuilder-options)
* [RequestOptions](#requestoptions)
* [TransitionOptions](#transitionoptions)
* [RequestBuilder](#requestbuilder)
    * [Picking a resource type](#picking-a-resource-type)
    * [Applying RequestOptions](#applying-requestoptions)
    * [Thumbnail requests](#thumbnail-requests)
    * [Starting a new request on failure.](#starting-a-new-request-on-failure)
* [Component Options](#component-options)

###  **RequestBuilder options**


Most options in Glide can be applied directly on the <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code> object returned by <code>Glide.with()</code>


Available options include (among others):

* Placeholders
* Transformations
* Caching Strategies
* Component specific options, like encode quality, or decode `Bitmap` configurations.

For example, to apply a <code>[CenterCrop](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html)</code> <code>[Transformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Transformation.html)</code>, you’d use the following:



```
Glide.with(fragment)
    .load(url)
    .centerCrop()
    .into(imageView);
```



### **RequestOptions**


If you’d like to share options consistently in loads across different parts of your app, you can also instantiate a new `RequestOptions` object and pass it in to each load with the <code>[apply()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#apply-com.bumptech.glide.request.RequestOptions-)</code> method.


```
RequestOptions cropOptions = new RequestOptions().centerCrop(context);
...
Glide.with(fragment)
    .load(url)
    .apply(cropOptions)
    .into(imageView);
```



<code>[apply()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#apply-com.bumptech.glide.request.RequestOptions-)</code> can be called multiple times, so <code>RequestOptions</code> can be composed. If two <code>RequestOptions</code> objects contain conflicting settings, the last <code>RequestOptions</code> applied to the load will win.


### **TransitionOptions**


[TransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/TransitionOptions.html) determine what will happen when your requested load completes.


 Use `TransitionOptions` to apply:



* View fade in
* Cross fade from the placeholder
* No transition

 Without a transition, your image will pop into place, immediately replacing the previous image. To avoid the sudden change, you can fade in your view, or cross fade between `Drawables` using `TransitionOptions`.


 For example, to apply a cross fade:



```
import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;

Glide.with(fragment)
    .load(url)
    .transition(withCrossFade())
    .into(view);
```



Unlike `RequestOptions`, `TransitionOptions` are type specific and are tied to the type of resource you ask Glide to load.


As a result, if you request a `Bitmap`, you will need to use [BitmapTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html), rather than [DrawableTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html). As a result, if you request a `Bitmap`, you may need to do a simple fade in, rather than a cross fade.


### **RequestBuilder**

[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html) is the backbone of the request in Glide and is responsible for bringing your options together with your requested url or model to start a new load.


Use `RequestBuilder` to specify:

* The type of resource you want to load (Bitmap, Drawable etc)
* The url/model you want to load the resource from
* The view you want to load the resource into
* Any `RequestOption` object(s) you want to apply
* Any `TransitionOption` object(s) you want to apply
* Any <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> you want to load.

You obtain a <code>RequestBuilder</code> by calling <code>Glide.with()</code> and then one of the <code>as</code> methods:

```
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
```


Or by calling `Glide.with()` and then `load()`:


```
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).load(url);
```

#### **Picking a resource type**

 `RequestBuilders` are specific to the type of resource they will load. By default you get a Drawable RequestBuilder. You can change the requested type using `as...` methods. For example, if you call `asBitmap()` you will get a `Bitmap` `RequestBuilder` instead:


```
RequestBuilder<Bitmap> requestBuilder = Glide.with(fragment).asBitmap();
```



#### **Applying RequestOptions**

As mentioned above, you apply `RequestOptions` with the <code>[apply()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#apply-com.bumptech.glide.request.RequestOptions-)</code> method and <code>TransitionOptions</code> with the <code>[transition()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#transition-com.bumptech.glide.TransitionOptions-)</code> method:


```
RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
requestBuilder.apply(requestOptions);
requestBuilder.transition(transitionOptions);
```


RequestBuilders can also be re-used to start multiple loads:


```
RequestBuilder<Drawable> requestBuilder =
    Glide.with(fragment)
      .asDrawable()
      .apply(requestOptions);

for (int i = 0; i < numViews; i++) {
   ImageView view = viewGroup.getChildAt(i);
   String url = urls.get(i);
   requestBuilder.load(url).into(view);
}
```



#### **Thumbnail requests**

Glide’s <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> API allows you to specify a <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code> to start in parallel with your main request. The <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> will be displayed while the primary request is loading. If the primary request completes before the thumbnail request, the image from the thumbnail request will not be shown. The <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> API allows you easily and quickly load low resolution versions of your images while your full quality equivalents load, reducing the amount of time users spend staring at loading indicators.


The <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> API is useful for both local and remote images, especially once the lower resolution thumbnails are in Glide’s disk cache where they can be loaded very quickly.


The <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> API is relatively simple to use:


```
Glide.with(fragment)
  .load(url)
  .thumbnail(
    Glide.with(fragment)
      .load(thumbnailUrl))
  .into(imageView);
```

This will work well if your `thumbnailUrl` points to a lower resolution image than your primary `url`. A number of image loading APIs offer ways for you to specify the size of the image you want in your URL, which works particularly well with the <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> API.


If you’re just loading a local image, or you only have a single remote URL, you can still benefit from the thumbnail API by using Glide’s <code>[override()](https://bumptech.github.io/glide/javadocs/430/com/bumptech/glide/request/RequestOptions.html#override-int-int-)</code> or <code>[sizeMultiplier()](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/request/RequestOptions.html#sizeMultiplier-float-)</code> APIs to force Glide to load a lower resolution image in the thumbnail request:


```
int thumbnailSize = ...;
Glide.with(fragment)
  .load(localUri)
  .thumbnail(
    Glide.with(fragment)
      .load(localUri)
      .override(thumbnailSize))
  .into(view);
```


 There’s a <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/430/com/bumptech/glide/RequestBuilder.html#thumbnail-float-)</code> convenience method that just takes a <code>sizeMultiplier</code> if you just want to load the same model at some percentage of your <code>View</code> or <code>Target</code>’s size:


```
Glide.with(fragment)
  .load(localUri)
  .thumbnail(/*sizeMultiplier=*/ 0.25f)
  .into(imageView);
```



####  **Starting a new request on failure.**

Starting in Glide 4.3.0, you can now easily specify a <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code> to use to start a new load if your main request fails using the <code>[error()](https://bumptech.github.io/glide/javadocs/430/com/bumptech/glide/RequestBuilder.html#error-com.bumptech.glide.RequestBuilder-)</code> API. For example, to load <code>fallbackUrl</code> if your request for <code>primaryUrl</code> fails:


```
Glide.with(fragment)
  .load(primaryUrl)
  .error(
      Glide.with(fragment)
        .load(fallbackUrl))
  .into(imageView);
```


The [error](https://bumptech.github.io/glide/javadocs/430/com/bumptech/glide/RequestBuilder.html#error-com.bumptech.glide.RequestBuilder-) <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code> will not be started if the main request completes successfully. If you specify both a <code>[thumbnail()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#thumbnail-com.bumptech.glide.RequestBuilder-)</code> and an <code>[error()](https://bumptech.github.io/glide/javadocs/430/com/bumptech/glide/RequestBuilder.html#error-com.bumptech.glide.RequestBuilder-)</code> <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code>, the error <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code> will be started if the primary request fails, even if the thumbnail request succeeds.


### **Component Options**

The <code>[Option](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Option.html)</code> class is a generic way of adding parameters to Glide’s components, including <code>[ModelLoaders](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/model/ModelLoader.html)</code>, <code>[ResourceDecoders](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/ResourceDecoder.html)</code>, <code>[ResourceEncoders](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/ResourceEncoder.html)</code>, <code>[Encoders](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Encoder.html)</code> etc. Some of Glide’s built in components contain Options and Options can be added to custom components as well”

```
Glide.with(context)
  .load(url)
  .option(MyCustomModelLoader.TIMEOUT_MS, 1000L)
  .into(imageView);
```

You can also create a new RequestOptions object:


```
RequestOptions options = 
    new RequestOptions()
      .set(MyCustomModelLoader.TIMEOUT_MS, 1000L);

Glide.with(context)
  .load(url)
  .apply(options)
  .into(imageView);
