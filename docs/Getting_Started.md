
## **Getting Started**

* [Basic Usage](#basic-usage)
* [Customizing requests.](#customizing-requests)
* [ListView and RecyclerView](#listview-and-recyclerview)
* [Non-View Targets](#non-view-targets)
* [Background Threads](#background-threads)

###  **Basic Usage**


Loading images with Glide is easy and in many cases requires only a single line:



```
Glide.with(fragment)
    .load(myUrl)
    .into(imageView);
```

 Cancelling loads you no longer need is simple too:


```
Glide.with(fragment).clear(imageView);
```


 Although it’s good practice to clear loads you no longer need, you’re not required to do so. In fact, Glide will automatically clear the load and recycle any resources used by the load when the Activity or Fragment you pass in to <code>[Glide.with()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/Glide.html#with-android.app.Fragment-)</code> is destroyed.


### **Customizing requests.**


Glide offers a variety of options that can be applied to individual requests, including transformations, transitions, caching options etc.


Default options can be applied directly on a request:



```
Glide.with(fragment)
  .load(myUrl)
  .placeholder(placeholder)
  .fitCenter()
  .into(imageView);
```



Options can be shared across requests using the `RequestOptions` class:


```
RequestOptions sharedOptions = 
    new RequestOptions()
      .placeholder(placeholder)
      .fitCenter();

Glide.with(fragment)
  .load(myUrl)
  .apply(sharedOptions)
  .into(imageView1);

Glide.with(fragment)
  .load(myUrl)
  .apply(sharedOptions)
  .into(imageView2);
```



Glide’s API can be further extended to include custom options using Glide’s [generated API](https://nickyshe.github.io/Glide-V4/#/Generated_API) for advanced use cases.


### **ListView and RecyclerView**


 Loading images in a ListView or RecyclerView uses the same load line as if you were loading in to a single View. Glide handles View re-use and request cancellation automatically:


```
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    String url = urls.get(position);
    Glide.with(fragment)
        .load(url)
        .into(holder.imageView);
}
```

 You don’t need to null check your urls either, Glide will either clear out the view or set whatever [placeholder Drawable](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#placeholder-int-) or [fallback Drawable](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#fallback-int-) you’ve specified if the url is null.


 Glide’s sole requirement is that any re-usable `View` or <code>[Target](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/target/Target.html)</code> that you may have started a load into at a previous position either has a new loaded started into it or is explicitly cleared via the <code>[clear()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> API.


```
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    if (isImagePosition(position)) {
        String url = urls.get(position);
        Glide.with(fragment)
            .load(url)
            .into(holder.imageView);
    } else {
        Glide.with(fragment).clear(holder.imageView);
        holder.imageView.setImageDrawable(specialDrawable);
    }
}
```



 By calling <code>[clear()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> or <code>into(View)</code> on the <code>View</code>, you’re cancelling the load and guaranteeing that Glide will not change the contents of the view after the call completes. If you forget to call <code>[clear()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html#clear-com.bumptech.glide.request.target.Target-)</code> and don’t start a new load, the load you started into the same View for a previous position may complete after you set your special <code>Drawable</code> and change the contents of the <code>View</code> to an old image.


 Although the examples we’ve shown here are for RecyclerView, the same principles apply to ListView as well.


### **Non-View Targets**


In addition to loading `Bitmap`s and `Drawable`s into `View`s, you can also start asynchronous loads into your own custom `Target`s:


```
Glide.with(context)
  .load(url)
  .into(new CustomTarget<Drawable>() {
    @Override
    public void onResourceReady(Drawable resource, Transition<Drawable> transition) {
      // Do something with the Drawable here.
    }

    @Override
    public void onLoadCleared(@Nullable Drawable placeholder) {
      // Remove the Drawable provided in onResourceReady from any Views and ensure 
      // no references to it remain.
    }
  });
```



There are a few gotchas with using custom `Target`s, so be sure to check out the [Targets docs page](https://nickyshe.github.io/Glide-V4/#/Targets) for details.


### **Background Threads**

Loading images on background threads is also straight forward using <code>[submit(int, int)](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/RequestBuilder.html#submit-int-int-)</code>:


```
FutureTarget<Bitmap> futureTarget =
  Glide.with(context)
    .asBitmap()
    .load(url)
    .submit(width, height);

Bitmap bitmap = futureTarget.get();

// Do something with the Bitmap and then when you're done with it:
Glide.with(context).clear(futureTarget);
```



 You can also start asynchronous loads on background threads the same way you would on a foreground thread if you don’t need the `Bitmap` or `Drawable` on the background thread itself:


```
Glide.with(context)
  .asBitmap()
  .load(url)
  .into(new Target<Bitmap>() {
    ...
  });
