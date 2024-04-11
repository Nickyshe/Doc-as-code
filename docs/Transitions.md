

##  **Transitions **



* [About](#about)
* [Default transition](#default-transition)
* [Standard behavior](#standard-behavior)
* [Specifying Transitions](#specifying-transitions)
* [Performance Tips](#performance-tips)
* [Common Errors](#common-errors)
    * [Cross fading with placeholders and transparent images](#cross-fading-with-placeholders-and-transparent-images)
    * [Cross fading across requests.](#cross-fading-across-requests)
* [Custom Transitions](#custom-transitions)

### **About**
<code>[Transitions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/Transition.html)</code> in Glide allow you to define how Glide should transition from a placeholder to a newly loaded image or from a thumbnail to a full size image. Transitions act within the context of a single request, not across multiple requests. As a result, <code>[Transitions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/Transition.html)</code> do <strong>NOT</strong> allow you to define an animation (like a cross fade) from one request to another request.


### **Default transition**

Unlike Glide v3, Glide v4 does **NOT** apply a cross fade or any other transition by default. Transitions must be applied manually per request.


###  **Standard behavior**


Glide provides a number of transitions that users can manually apply per request. Glide’s built in transitions behave in a consistent manner and will avoid running in certain circumstances depending on where images are loaded from.


Images can be loaded from one of four places in Glide:

1. Glide’s in memory cache
2. Glide’s disk cache
3. A source File or Uri available locally on the device
4. A source Url or Uri available only remotely.

Glide’s built in transitions do not run if data is loaded from Glide’s in memory cache. However, Glide’s built in transitions do run if data is loaded from Glide’s disk cache, a local source File or Uri or a remote source Url or Uri.


To change this behavior and write your own custom transition, see the [custom transitions](https://bumptech.github.io/glide/transitions#custom-transitions) section below.


### **Specifying Transitions**


For an overview and code sample, see the [Options documentation](https://bumptech.github.io/glide/doc/options.html#transitionoptions).


<code>[TransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/TransitionOptions.html)</code> are used to specify the transitions for a particular request. <code>[TransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/TransitionOptions.html)</code> are set for a request using the <code>[transition()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html#transition-com.bumptech.glide.TransitionOptions-)</code> method in <code>[RequestBuilder](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestBuilder.html)</code>. Type specific transitions can be specified using <code>[BitmapTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/BitmapTransitionOptions.html)</code> or <code>[DrawableTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html)</code>. For types other than <code>Bitmaps</code> and <code>Drawables</code> <code>[GenericTransitionOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/GenericTransitionOptions.html)</code> can be used.


###  **Performance Tips**

 Animations in Android can be expensive, particularly if a large number are started at once. Cross fades and other animations involving changes in alpha can be especially expensive. In addition, animations often take substantially longer to run than images take to decode. Gratuitous use of animations in lists and grids can make image loading feel slow and janky. To maximize performance, consider avoiding animations when using Glide to load images into ListViews, GridViews, or RecyclerViews, especially when you expect images to be cached or fast to load most of the time. Instead consider pre-loading so that images are in memory when users scroll to them.


### **Common Errors**


####  **Cross fading with placeholders and transparent images**

Glide’s default cross fade animation leverages <code>[TransitionDrawable](https://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html)</code>. <code>[TransitionDrawable](https://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html)</code> offers two animation modes, controlled by <code>[setCrossFadeEnabled()](https://developer.android.com/reference/android/graphics/drawable/TransitionDrawable.html#setCrossFadeEnabled(boolean))</code>. When cross fades are disabled, the image that is transitioned to is faded in on top of the image that was already showing. When cross fades are enabled, the image that is being transitioned from is animated from opaque to transparent and the image that is being transitioned to is animated from transparent to opaque.


In Glide, we default to disabling cross fades because it typically provides a much nicer looking animation. An actual cross fade where the alpha of both images is changing at once often produces a white flash in the middle of the animation where both images are partially opaque.


Unfortunately although disabling cross fades is typically a better default, it can also lead to problems when the image that is being loaded contains transparent pixels. When the placeholder is larger than the image that is being loaded or the image is partially transparent, disabling cross fades results in the placeholder being visible behind the image after the animation finishes. If you are loading transparent images with placeholders, you can enable cross fades by adjusting the options in <code>[DrawableCrossFadeFactory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/DrawableCrossFadeFactory.html)</code> and passing the result into <code>[transition()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/TransitionOptions.html#transition-com.bumptech.glide.request.transition.TransitionFactory-)</code>:



```
DrawableCrossFadeFactory factory =
        new DrawableCrossFadeFactory.Builder().setCrossFadeEnabled(true).build();

GlideApp.with(context)
        .load(url)
        .transition(withCrossFade(factory))
        .diskCacheStrategy(DiskCacheStrategy.ALL)
        .placeholder(R.color.placeholder)
        .into(imageView);
```



See [Issue #2017](https://github.com/bumptech/glide/issues/2017) for more information. Thanks @minas90 for writing down the example above.


#### **Cross fading across requests.**


<code>[Transitions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/Transition.html)</code> do not allow you to cross fade between two different images that are loaded with different requests. Glide by default will cancel any existing requests when you start a new load into an existing View or Target (See [Targets documentation](https://bumptech.github.io/glide/doc/targets.html#targets-and-automatic-cancellation) for more details). As a result, if you want to load two different images and cross fade between them, you cannot do so with Glide directly. Strategies like waiting for the first load to finish, grabbing a Bitmap or Drawable out of the View, starting a second load, and then manually animating between the Drawale or Bitmap and the new image are unsafe and may result in crashes or graphical corruption.


Instead, the easiest way to cross fade across two different images loaded in two separate requests is to use <code>[ViewSwitcher](https://developer.android.com/reference/android/widget/ViewSwitcher.html)</code> containing two <code>[ImageViews](https://developer.android.com/reference/android/widget/ImageView.html)</code>. Load the first image into the result of <code>[getNextView()](https://developer.android.com/reference/android/widget/ViewSwitcher.html#getNextView())</code>. Then load the second image into the next result of <code>[getNextView()](https://developer.android.com/reference/android/widget/ViewSwitcher.html#getNextView())</code> and use a <code>[RequestListener](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestListener.html)</code> to call <code>[showNext()](https://developer.android.com/reference/android/widget/ViewAnimator.html#showNext())</code> when the second image load finishes. For better control, you can also follow the strategy outlined in the [developer documentation](https://developer.android.com/training/animation/crossfade.html). As with the <code>[ViewSwitcher](https://developer.android.com/reference/android/widget/ViewSwitcher.html)</code>, only start the cross fade after the second image load finishes.


### **Custom Transitions**

To define a custom transition:



1. Implement <code>[TransitionFactory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/TransitionFactory.html)</code>.
2. Apply your custom <code>TransitionFactory</code> to loads with <code>[DrawableTransitionOptions#with](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/drawable/DrawableTransitionOptions.html#with-com.bumptech.glide.request.transition.TransitionFactory-)</code>.

To change the default behavior of your transition so that you can control whether or not it’s applied when your image is loaded from the memory cache, disk cache or from source, you can inspect the <code>[DataSource](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/DataSource.html)</code> passed in to the <code>[build()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/TransitionFactory.html#build-com.bumptech.glide.load.DataSource-boolean-)</code> method in your <code>[TransitionFactory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/transition/TransitionFactory.html)</code>,


For an example, see <code>[DrawableCrossFadeFactory](https://github.com/bumptech/glide/blob/8f22bd9b82349bf748e335b4a31e70c9383fb15a/library/src/main/java/com/bumptech/glide/request/transition/DrawableCrossFadeFactory.java#L35)</code>.
