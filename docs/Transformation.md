
## **Transformation**
* [About](#about)
* [Built in types](#built-in-types)
* [Applying Transformations](#applying-transformations)
    * [Default Transformations](#default-transformations)
    * [Multiple Transformations.](#multiple-transformations)
* [Custom transformations.](#custom-transformations)
    * [BitmapTransformation](#bitmaptransformation)
    * [Required methods](#required-methods)
    * [Don’t forget equals()/hashCode()!](#dont-forget-equalshashcode)
* [Special Behavior in Glide](#special-behavior-in-glide)
    * [Re-using Transformations](#re-using-transformations)
    * [Automatic Transformations for ImageViews](#automatic-transformations-for-imageviews)
    * [Custom resources](#custom-resources)

###  **About**


[Transformations](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Transformation.html) in Glide take a resource, mutate it, and return the mutated resource. Typically transformations are used to crop, or apply filters to Bitmaps, but they can also be used to transform animated GIFs, or even custom resource types.


### **Built in types**


Glide includes a number of built in transformations, including:

* [CenterCrop](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html)
* [FitCenter](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html)
* [CircleCrop](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/CircleCrop.html)

### **Applying Transformations**


Transformations are applied using the [RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html) class:


#### **Default Transformations**



```
Glide.with(fragment)
  .load(url)
  .fitCenter()
  .into(imageView);
```



Or with `RequestOptions`:


```
RequestOptions options = new RequestOptions();
options.centerCrop();

Glide.with(fragment)
    .load(url)
    .apply(options)
    .into(imageView);
```

For more information on using RequestOptions, see the [Options](https://nickyshe.github.io/Doc-as-code/#/Options) wiki page.


#### **Multiple Transformations.**


By default, each subsequent call to <code>[transform()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#transform-java.lang.Class-com.bumptech.glide.load.Transformation-)</code> or any specific transform method (<code>fitCenter()</code>, <code>centerCrop()</code>, <code>bitmapTransform()</code> etc) will replace the previous transformation.

 To instead apply multiple transformations to a single load, use the <code>[MultiTransformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/MultiTransformation.html)</code> class or the shortcut <code>[.transforms()](https://bumptech.github.io/glide/javadocs/410/com/bumptech/glide/request/RequestOptions.html#transforms-com.bumptech.glide.load.Transformation...-)</code> method.


```
Glide.with(fragment)
  .load(url)
  .transform(new MultiTransformation(new FitCenter(), new YourCustomTransformation())
  .into(imageView);
```



Or with the shortcut method:


```
Glide.with(fragment)
  .load(url)
  .transform(new FitCenter(), new YourCustomTransformation())
  .into(imageView);
```



The order in which you pass transformations to <code>[MultiTransformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/MultiTransformation.html)</code>’s constructor determines the order in which the transformations are applied.


### **Custom transformations.**


Although Glide provides a variety of built in <code>[Transformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Transformation.html)</code> implementations, you may also implement your own custom <code>[Transformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html)</code>s if you need additional functionality.


#### **BitmapTransformation**

If you only need to transform `Bitmap`s, it’s best to start by subclassing <code>[BitmapTransformation](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html)</code>. <code>BitmapTransformation</code> takes care of a few of the basic things for you, including extracting recycling the original Bitmap if your <code>Transformation</code> returns a new modified Bitmap.


A simple implementation might look something like this:


```
public class FillSpace extends BitmapTransformation {
    private static final String ID = "com.bumptech.glide.transformations.FillSpace";
    private static final byte[] ID_BYTES = ID.getBytes(Charset.forName("UTF-8"));

    @Override
    public Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        if (toTransform.getWidth() == outWidth && toTransform.getHeight() == outHeight) {
            return toTransform;
        }

        return Bitmap.createScaledBitmap(toTransform, outWidth, outHeight, /*filter=*/ true);
    }

    @Override
    public boolean equals(Object o) {
      return o instanceof FillSpace;
    }

    @Override
    public int hashCode() {
      return ID.hashCode();
    }

    @Override
    public void updateDiskCacheKey(MessageDigest messageDigest) {
      messageDigest.update(ID_BYTES);
    }
}
```



Although your `Transformation` will almost certainly do something more sophisticated than our example, it should contain the same basic elements and method overrides.


#### **Required methods**


In particular, note that there are three methods that you **must** implement for any `Transformation` subclass, including `BitmapTransformation` in order for disk and memory caching to work correctly:



1. `equals()`
2. `hashCode()`
3. `updateDiskCacheKey`

If your <code>[Transformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Transformation.html)</code> takes no arguments, it’s often easy to just use a <code>static</code> <code>final</code> <code>String</code> containing the fully qualified package name as an id that can form the basis of <code>hashCode()</code> and can be used to update the <code>MessageDigest</code> passed to <code>updateDiskCacheKey()</code>. If your <code>[Transformation](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Transformation.html)</code> does take arguments that affect the way the <code>Bitmap</code> is transformed, they must be included in these three methods as well.


For example, Glide’s <code>[RoundedCorners](https://bumptech.github.io/glide/javadocs/440/com/bumptech/glide/load/resource/bitmap/RoundedCorners.html)</code> <code>Transformation</code> accepts an <code>int</code> that determines the radius of the rounded corners. Its implementations of <code>equals()</code>, <code>hashCode()</code> and <code>updateDiskCacheKey</code> looks like this:



```
 @Override
  public boolean equals(Object o) {
    if (o instanceof RoundedCorners) {
      RoundedCorners other = (RoundedCorners) o;
      return roundingRadius == other.roundingRadius;
    }
    return false;
  }

  @Override
  public int hashCode() {
    return Util.hashCode(ID.hashCode(),
        Util.hashCode(roundingRadius));
  }

  @Override
  public void updateDiskCacheKey(MessageDigest messageDigest) {
    messageDigest.update(ID_BYTES);

    byte[] radiusData = ByteBuffer.allocate(4).putInt(roundingRadius).array();
    messageDigest.update(radiusData);
  }
```



The original `String` id remains as well, but the `roundingRadius` is included in all three methods as well. The `updateDiskCacheKey` method here also demonstrates how you can use `ByteBuffer` to including primitive arguments in your `updateDiskCacheKey` implementation.


#### **Don’t forget equals()/hashCode()!**


It’s worth re-iterating one more time that it’s essential that you implement `equals()` and `hashCode()` for memory caching to work correctly. Unfortunately `BitmapTransformation` and `Transformation` implementations will compile if those methods are not overridden, but that doesn’t mean they work correctly. We’re exploring options for making using the default `equals()` and `hashCode` methods a compile time error in future versions of Glide.


### **Special Behavior in Glide**


#### **Re-using Transformations**


`Transformations` are meant to be stateless. As a result, it should always be safe to re-use a `Transformation` instance for multiple loads. It’s usually good practice to create a `Transformation` once and then pass it in to multiple loads.


#### **Automatic Transformations for ImageViews**


When you start a load into an [ImageView](https://developer.android.com/reference/android/widget/ImageView.html) in Glide, Glide may automatically apply either [FitCenter](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/FitCenter.html) or [CenterCrop](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/bitmap/CenterCrop.html), depending on the [ScaleType](https://developer.android.com/reference/android/widget/ImageView.ScaleType.html) of the view. If the scale type is `CENTER_CROP`, Glide will automatically apply the `CenterCrop` transformation. If the scale type is `FIT_CENTER` or `CENTER_INSIDE`, Glide will automatically apply the `FitCenter` transformation.


You can always override the default transformation by applying a [RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html) with a `Transformation` set. In addition, you can ensure no `Transformation` is automatically applied using <code>[dontTransform()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#dontTransform--)</code>.


#### **Custom resources**


Because Glide 4.0 allows you to specify a super type of the resource you’re going to decode, you may not know exactly what type of transformation to apply. For example, when you use <code>[asDrawable()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/RequestManager.html#asDrawable--)</code> (or just <code>with()</code> since <code>asDrawable()</code> is the default) to ask for a Drawable resource, you may get either the <code>[BitmapDrawable](https://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html)</code> subclass, or the <code>[GifDrawable](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/resource/gif/GifDrawable.html)</code> subclass.


To ensure any `Transformation` you add to your `RequestOptions` is applied, Glide adds your `Transformation` to a map keyed on the resource class you provide to <code>[transform()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#transform-java.lang.Class-com.bumptech.glide.load.Transformation-)</code>. When a resource is successfully decoded , Glide uses the map to retrieve a corresponding <code>Transformation</code>.


Glide can apply `Bitmap` `Transformations` to `BitmapDrawable`, `GifDrawable`, and `Bitmap` resources, so typically you only need to write and apply `Bitmap` `Transformations`. However, if you add additional resource types you may need to consider sub-classing <code>[RequestOptions](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html)</code> and always applying a <code>Transformation</code> for your custom resource type in addition to the built in <code>Bitmap</code> <code>Transformations</code>.
