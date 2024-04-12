
## **RecyclerView**



* [About](#about)
* [Gradle](#gradle)
* [Setup](#setup)
    * [PreloadSizeProvider](#preloadsizeprovider)
    * [PreloadModelProvider](#preloadmodelprovider)
    * [RecyclerViewPreloader](#recyclerviewpreloader)
        * [maxPreload](#maxpreload)
    * [RecyclerView](#recyclerview)
    * [All together](#all-together)
* [Examples](#examples)
* [Tips and tricks](#tips-and-tricks)
* [Code](#code)


### **About**

The RecyclerView integration library makes the <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> available in your application. <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> can automatically load images just ahead of where a user is scrolling in a RecyclerView.

Combined with the right image size and an effective disk cache strategy, this library can dramatically decrease the number of loading tiles/indicators users see when scrolling through lists of images by ensuring that the images the user is about to reach are already in memory.


### **Gradle**

To use the RecyclerView integration library, add a dependency on it in your `build.gradle` file:


```
implementation ("com.github.bumptech.glide:recyclerview-integration:4.14.2") {
  // Excludes the support library because it's already included by Glide.
  transitive = false
}
```


If you haven’t already, you will also need to make sure that you already have a dependency on `RecyclerView` and that you’re using `RecyclerView` in your app :).


### **Setup**

To use the `RecyclerView` integration library you need to follow a couple of steps:



1. Create a <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code>
2. Create a <code>[PreloadModelProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/ListPreloader.PreloadModelProvider.html)</code>
3. Create the <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> given the <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code> and <code>[PreloadModelProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/ListPreloader.PreloadModelProvider.html)</code>s you created in the first two steps
4. Add your <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> to your <code>RecyclerView</code> as a scroll listener.

Each of these steps is outlined in more detail below.


#### **PreloadSizeProvider**

After you add the gradle dependency, you next need to create a <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code>. The <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code> is responsible for making sure your <code>RecyclerViewPreloader</code> loads images in the same size as those loaded by your adapters <code>onBindViewHolder</code> method.

Glide provides two built in implementations of <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code>:



1. <code>[ViewPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/util/ViewPreloadSizeProvider.html)</code>
2. <code>[FixedPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/util/FixedPreloadSizeProvider.html)</code>

If you have uniform <code>View</code> sizes in your <code>RecyclerView</code>, you’re loading images with <code>into(ImageView)</code> and you’re not using <code>override()</code> to set a different size, you can use <code>[ViewPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/util/ViewPreloadSizeProvider.html)</code>.

If you’re using `override()` or are otherwise loading image sizes that don’t exactly match the size of your `Views`, you can use <code>[FixedPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/util/FixedPreloadSizeProvider.html)</code>.

If the logic required to determine the image size used for a given position in your `RecyclerView` doesn’t fit either of those cases, you can always write your own implementation of <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code>.

If you are using a fixed size to load your images, typically <code>[FixedPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/util/FixedPreloadSizeProvider.html)</code> is simplest:


```
private final imageWidthPixels = 1024;
private final imageHeightPixels = 768;

...

PreloadSizeProvider sizeProvider = 
    new FixedPreloadSizeProvider(imageWidthPixels, imageHeightPixels);
```



#### **PreloadModelProvider**

The next step is to implement your <code>[PreloadModelProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/ListPreloader.PreloadModelProvider.html)</code>. The <code>[PreloadModelProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/ListPreloader.PreloadModelProvider.html)</code> performs two actions. First it collects and returns a list of <code>Models</code> (the items you pass in to Glide’s <code>load(Object)</code> method, like URLs or file paths) for a given position. Second it takes a <code>Model</code> and produces a Glide <code>RequestBuilder</code> that will be used to preload the given <code>Model</code> into memory.

For example, let’s say that we have a `RecyclerView` that contains a list of image urls where each position in the `RecyclerView` displays a single URL. Then, let’s say that you load your images in your `RecyclerView.Adapter`’s `onBindViewHolder` method like this:


```
private List<String> myUrls = ...;

...

@Override
public void onBindViewHolder(ViewHolder viewHolder, int position) {
  ImageView imageView = ((MyViewHolder) viewHolder).imageView;
  String currentUrl = myUrls.get(position);

  Glide.with(fragment)
    .load(currentUrl)
    .override(imageWidthPixels, imageHeightPixels)
    .into(imageView);
}
```


Your <code>[PreloadModelProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/ListPreloader.PreloadModelProvider.html)</code> implementation might then look like this:


```
private List<String> myUrls = ...;

...

private class MyPreloadModelProvider implements PreloadModelProvider {
  @Override
  @NonNull
  List<U> getPreloadItems(int position) {
    String url = myUrls.get(position);
    if (TextUtils.isEmpty(url)) {
      return Collections.emptyList();
    }
    return Collections.singletonList(url);
  }

  @Override
  @Nullable
  RequestBuilder getPreloadRequestBuilder(String url) {
    return 
      Glide.with(fragment)
        .load(url) 
        .override(imageWidthPixels, imageHeightPixels);
  }
}
```


It’s critical that the `RequestBuilder` returned from `getPreloadRequestBuilder` use exactly the same set of options (placeholders, transformations etc) and exactly the same size as the request you start in `onBindViewHolder`. If any of the options aren’t exactly the same in the two methods for a given position, your preload request will be wasted because the image it loads will be cached with a cache key that doesn’t match the cache key of the image you load in `onBindViewHolder`. If you have trouble getting these cache keys to match, see the [debugging page](https://nickyshe.github.io/Glide-V4/#/Debugging#unexpected-cache-misses).

If you have nothing to preload for a given position, you can return an empty list from `getPreloadItems`. If you later discover that you’re unable to create a `RequestBuilder` for a given `Model`, you may return `null` from `getPreloadRequestBuilder`.


#### **RecyclerViewPreloader**

Once you have your <code>[PreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/ListPreloader.PreloadSizeProvider.html)</code> and your <code>[PreloadModelProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/ListPreloader.PreloadModelProvider.html)</code>, you’re ready to create your <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code>:


```
private final imageWidthPixels = 1024;
private final imageHeightPixels = 768;
private List<String> myUrls = ...;


...

PreloadSizeProvider sizeProvider = 
    new FixedPreloadSizeProvider(imageWidthPixels, imageHeightPixels);
PreloadModelProvider modelProvider = new MyPreloadModelProvider();
RecyclerViewPreloader<Photo> preloader = 
    new RecyclerViewPreloader<>(
        Glide.with(this), modelProvider, sizeProvider, 10 /*maxPreload*/);
```


Using 10 for maxPreload is just a placeholder, for a detailed discussion on how to pick a number, see the section immediately below this one.


##### **maxPreload**

The `maxPreload` is an integer that indicates how many items you want to preload. The optimal number will vary by your image size, quantity, the layout of your `RecyclerView` and in some cases even the devices your application is running on.

A good starting point is to pick a number large enough to include all of the images in two or three rows. Once you’ve picked your initial number, you can try running your application on a couple of devices and tweaking it as necessary to maximize the number of cache hits.

An overly large number will mean you’re preloading too far ahead to be useful. An overly small number will prevent you from loading enough images ahead of time.


#### **RecyclerView**

The final step, once you have your <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> is to add it as a scroll listener to your <code>RecyclerView</code>:


```
RecyclerView myRecyclerView = (RecyclerView) findViewById(R.id.recycler_view);
myRecyclerView.addOnScrollListener(preloader);
```


Adding the <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> as a scroll listener allows the <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> to automatically load images ahead of the direction the user is scrolling in and detect changes of direction or velocity.

**Warning** - Glide’s default scroll listener, <code>[RecyclerToListViewScrollListener](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerToListViewScrollListener.html)</code> assumes you’re using a <code>[LinearLayoutManager](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html)</code> or a subclass and will crash if that’s not the case. If you’re using a different <code>LayoutManager</code> type, you will need to implement your own <code>[OnScrollListener](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.OnScrollListener.html)</code>, translate the calls <code>RecyclerView</code> provides into positions, and call <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code> with those positions.


#### **All together**

Once you’ve completed all of these steps, you’ll end up with something like this:


```
public final class ImagesFragment extends Fragment {
  // These are totally arbitrary, pick sizes that are right for your UI.
  private final imageWidthPixels = 1024;
  private final imageHeightPixels = 768;
  // You will need to populate these urls somewhere...
  private List<String> myUrls = ...;

  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container,
      Bundle savedInstanceState) {


    View result = inflater.inflate(R.layout.images_fragment, container, false);

    PreloadSizeProvider sizeProvider = 
        new FixedPreloadSizeProvider(imageWidthPixels, imageHeightPixels);
    PreloadModelProvider modelProvider = new MyPreloadModelProvider();
    RecyclerViewPreloader<Photo> preloader = 
        new RecyclerViewPreloader<>(
            Glide.with(this), modelProvider, sizeProvider, 10 /*maxPreload*/);

    RecyclerView myRecyclerView = (RecyclerView) result.findViewById(R.id.recycler_view);
    myRecyclerView.addOnScrollListener(preloader);


    // Finish setting up your RecyclerView etc.
    myRecylerView.setLayoutManager(...);
    myRecyclerView.setAdapter(...);

    ... 

    return result;
  }

  private class MyPreloadModelProvider implements PreloadModelProvider {
    @Override
    @NonNull
    public List<U> getPreloadItems(int position) {
      String url = myUrls.get(position);
      if (TextUtils.isEmpty(url)) {
        return Collections.emptyList();
      }
      return Collections.singletonList(url);
    }

    @Override
    @Nullable
    public RequestBuilder getPreloadRequestBuilder(String url) {
      return 
        Glide.with(fragment)
          .load(url) 
          .override(imageWidthPixels, imageHeightPixels);
    }
  }
}
```



### **Examples**

Glide’s [sample apps](https://bumptech.github.io/glide/ref/samples.html) contain a couple of example usages of <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code>, including:



1. [FlickrPhotoGrid](https://github.com/bumptech/glide/blob/853c0d94f1ad353048b3d2556b49729ef3534430/samples/flickr/src/main/java/com/bumptech/glide/samples/flickr/FlickrPhotoGrid.java#L107), uses a <code>[FixedPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/util/FixedPreloadSizeProvider.html)</code> to preload in the flickr sample’s two smaller photo grid views.
2. [FlickrPhotoList](https://github.com/bumptech/glide/blob/853c0d94f1ad353048b3d2556b49729ef3534430/samples/flickr/src/main/java/com/bumptech/glide/samples/flickr/FlickrPhotoList.java#L68) uses a <code>[ViewPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/util/ViewPreloadSizeProvider.html)</code> to preload in the flickr sample’s larger list view.
3. [MainActivity](https://github.com/bumptech/glide/blob/853c0d94f1ad353048b3d2556b49729ef3534430/samples/giphy/src/main/java/com/bumptech/glide/samples/giphy/MainActivity.java#L48) in the Giphy sample uses a <code>[ViewPreloadSizeProvider](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/util/ViewPreloadSizeProvider.html)</code> to preload GIFs while scrolling.
4. [HorizontalGalleryFragment](https://github.com/bumptech/glide/blob/853c0d94f1ad353048b3d2556b49729ef3534430/samples/gallery/src/main/java/com/bumptech/glide/samples/gallery/HorizontalGalleryFragment.java#L53) in the Gallery sample uses a custom <code>PreloadSizeProvider</code> to preload local images while scrolling horizontally.


### <strong>Tips and tricks</strong>



1. Use `override()` to ensure that the images you’re loading are uniformally sized in your `Adapter` and in your <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code>. You don’t necessarily need to match the size of the <code>View</code> you’re using exactly with the dimensions you pass in to <code>override()</code>, Android’s <code>ImageView</code> class can easily handle scaling up or down minor differences in sizes.
2. Fewer larger images are often faster to load than many smaller images. There’s a fair amount of overhead to starting each request, so if you can, load fewer and larger images in your UI.
3. If scrolling is janky, consider using <code>override()</code> to deliberately decrease image sizes. Uploading textures (Bitmaps) on Android can be expensive, especially for large Bitmaps. You can use <code>override()</code> to force the images to be smaller than your <code>Views</code> for smoother scrolling. You can even swap out the lower resolution images with higher resolution images once the user stops scrolling if you’re concerned about quality.
4. Check out the [unexpected cache misses](https://bumptech.github.io/glide/doc/debugging.html#unexpected-cache-misses) section of the debugging docs page if you’re having trouble getting your <code>Adapter</code> to use the images loaded in your <code>[RecyclerViewPreloader](https://bumptech.github.io/glide/javadocs/420/com/bumptech/glide/integration/recyclerview/RecyclerViewPreloader.html)</code>


### <strong>Code</strong>



* [https://github.com/bumptech/glide/tree/master/integration/recyclerview](https://github.com/bumptech/glide/tree/master/integration/recyclerview)