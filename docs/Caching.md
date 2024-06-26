
##  **Caching**
* [Caching in Glide](#caching-in-glide)
* [Cache Keys](#cache-keys)
* [Cache Configuration](#cache-configuration)
    * [Disk Cache Strategies](#disk-cache-strategies)
    * [Loading only from cache](#loading-only-from-cache)
    * [Skipping the cache.](#skipping-the-cache)
    * [Implementation](#implementation)
* [Cache Invalidation](#cache-invalidation)
    * [Custom Cache Invalidation](#custom-cache-invalidation)
* [Resource Management](#resource-management)
    * [Memory Cache](#memory-cache)
        * [Permanent size changes](#permanent-size-changes)
        * [Temporary size changes.](#temporary-size-changes)
        * [Clearing memory](#clearing-memory)
    * [Disk Cache](#disk-cache)
        * [Permanent size changes](#permanent-size-changes-1)
        * [Clearing the disk cache](#clearing-the-disk-cache)

### **Caching in Glide**

By default, Glide checks multiple layers of caches before starting a new request for an image:
1. Active resources - Is this image displayed in another View right now?
2. Memory cache - Was this image recently loaded and still in memory?
3. Resource - Has this image been decoded, transformed, and written to the disk cache before?
4. Data - Was the data this image was obtained from written to the disk cache before?

    The first two steps check to see if the resource is in memory and if so, return the image immediately. The second two steps check to see if the image is on disk and return quickly, but asynchronously.


    If all four steps fail to find the image, then Glide will go back to the original source to retrieve the data (the original File, Uri, Url etc).


    For details on default sizes and locations of Glide’s caches or to configure those parameters, see the [configuration](https://bumptech.github.io/glide/doc/configuration.html#disk-cache) page.


### **Cache Keys**

In Glide 4, all cache keys contain at least two elements:

1. The model the load is requested for (File, Uri, Url). If you are using a custom model, it needs to correctly implements `hashCode()` and `equals()`
2. An optional <code>[Signature](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#signature-com.bumptech.glide.load.Key-)</code>

In fact, the cache keys for steps 1-3 (Active resources, memory cache, resource disk cache) also include a number of other pieces of data including:

1. The width and height
2. The optional `Transformation`
3. Any added <code>[Options](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Option.html)</code>
4. The requested data type (Bitmap, GIF, etc)

The keys used for active resources and the memory cache also differ slightly from those used from the resource disk cache to accomodate in memory <code>[Options](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Option.html)</code> like those thataffect the configuration of the Bitmap or other decode time only parameters.


To generate the name of disk cache keys on disk, the individual elements of the keys are hashed to create a single String key, which is then used as the file name in the disk cache.


### **Cache Configuration**


 Glide provides a number of options that allow you to choose how loads will interact with Glide’s caches on a per request basis.


#### **Disk Cache Strategies**

<code>[DiskCacheStrategy](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html)</code> can be applied with the <code>[diskCacheStrategy](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#diskCacheStrategy-com.bumptech.glide.load.engine.DiskCacheStrategy-)</code> method to an individual request. The available strategies allow you to prevent your load from using or writing to the disk cache or choose to cache only the unmodified original data backing your load, only the transformed thumbnail produced by your load, or both.


The default strategy, <code>[AUTOMATIC](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html#AUTOMATIC)</code>, tries to use the optimal strategy for local and remote images. <code>AUTOMATIC</code> will store only the unmodified data backing your load when you’re loading remote data (like from URLs) because downloading remote data is expensive compared to resizing data already on disk. For local data <code>AUTOMATIC</code> will store the transformed thumbnail only because retrieving the original data is cheap if you need to generate a second thumbnail size or type.


To apply a <code>[DiskCacheStrategy](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html)</code>:



```
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.ALL)
  .into(imageView);
```



#### **Loading only from cache**

In some circumstances you may want a load to fail if an image is not already in cache. To do so, you can use the <code>[onlyRetrieveFromCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#onlyRetrieveFromCache-boolean-)</code> method on a per request basis:


```
Glide.with(fragment)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView);
```



If the image is found in the memory cache or in the disk cache, it will be loaded. Otherwise, if this option is set to true, the load will fail.


#### **Skipping the cache.**

If you’d like to make sure a particular request skips either the disk cache or the memory cache or both, Glide provides a few alternatives.

To skip the memory cache only, use <code>[skipMemoryCache()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#skipMemoryCache-boolean-)</code>:


```
Glide.with(fragment)
  .load(url)
  .skipMemoryCache(true)
  .into(view);
```


To skip the disk cache only, use <code>[DiskCacheStrategy.NONE](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html#NONE)</code>:


```
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .into(view);
```



These options can be used together:


```
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .skipMemoryCache(true)
  .into(view);
```


In general you want to try to avoid skipping caches. It’s vastly faster to load an image from cache than it is to retrieve, decode, and transform it to create a new thumbnail.


If you’d just like to update the entry for an item in the cache, see the documentation on <code>[invalidation](https://bumptech.github.io/glide/doc/caching.html#cache-invalidation)</code> below.


#### **Implementation**

If the available options aren’t sufficient for your needs, you can also write your own <code>[DiskCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/cache/DiskCache.html)</code> implementation. See the [configuration](https://nickyshe.github.io/Glide-V4/#/Configurations#disk-cache) page for details.


### **Cache Invalidation**

Because disk cache are hashed keys, there is no good way to simply delete all of the cached files on disk that correspond to a particular url or file path. The problem would be simpler if you were only ever allowed to load or cache the original image, but since Glide also caches thumbnails and provides various transformations, each of which will result in a new File in the cache, tracking down and deleting every cached version of an image is difficult.

In practice, the best way to invalidate a cache file is to change your identifier when the content changes (url, uri, file path etc) when possible.


#### **Custom Cache Invalidation**

Since it’s often difficult or impossible to change identifiers, Glide also offers the <code>[signature()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#signature-com.bumptech.glide.load.Key-)</code> API to mix in additional data that you control into your cache key. Signatures work well for media store content, as well as any content you can maintain some versioning metadata for.



* Media store content - For media store content, you can use Glide’s <code>[MediaStoreSignature](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/signature/MediaStoreSignature.html)</code> class as your signature. <code>MediaStoreSignature</code> allows you to mix the date modified time, mime type, and orientation of a media store item into the cache key. These three attributes reliably catch edits and updates allowing you to cache media store thumbs.
* Files - You can use <code>[ObjectKey](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/signature/ObjectKey.html)</code> to mix in the File’s date modified time.
* Urls - Although the best way to invalidate urls is to make sure the server changes the url and updates the client when the content at the url changes, you can also use <code>[ObjectKey](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/signature/ObjectKey.html)</code> to mix in arbitrary metadata (such as a version number) instead.

Passing in signatures to loads is simple:



```
Glide.with(yourFragment)
    .load(yourFileDataModel)
    .signature(new ObjectKey(yourVersionMetadata))
    .into(yourImageView);
```


The media store signature is also straightforward if you have previously bulk loaded the necessary data from the MediaStore:


```
Glide.with(fragment)
    .load(mediaStoreUri)
    .signature(new MediaStoreSignature(mimeType, dateModified, orientation))
    .into(view);
```


You can also define your own signature by implementing the <code>[Key](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/Key.html)</code> interface. Be sure to implement <code>equals()</code>, <code>hashCode()</code> and the <code>updateDiskCacheKey()</code> method:


```
public class IntegerVersionSignature implements Key {
    private int currentVersion;

    public IntegerVersionSignature(int currentVersion) {
         this.currentVersion = currentVersion;
    }


    @Override
    public boolean equals(Object o) {
        if (o instanceof IntegerVersionSignature) {
            IntegerVersionSignature other = (IntegerVersionSignature) o;
            return currentVersion == other.currentVersion;
        }
        return false;
    }


    @Override
    public int hashCode() {
        return currentVersion;
    }

    @Override
    public void updateDiskCacheKey(MessageDigest md) {
        messageDigest.update(ByteBuffer.allocate(Integer.SIZE).putInt(signature).array());
    }
}
```


Keep in mind that to avoid degrading performance, you will want to batch load any versioning metadata in the background so that it is available when you want to load your image.

If all else fails and you can neither change your identifier nor keep track of any reasonable version metadata, you can also disable disk caching entirely using <code>[diskCacheStrategy()](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/request/RequestOptions.html#diskCacheStrategy-com.bumptech.glide.load.engine.DiskCacheStrategy-)</code> and <code>[DiskCacheStrategy.NONE](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/DiskCacheStrategy.html#NONE)</code>.


### **Resource Management**

Glide’s disk and memory caches are LRU which means that they take increasingly more memory and/or disk space until they reach their limit at which point they will use at or near the limit continuously. For some added flexibility, Glide provides a few additional ways you can manage the resources your application uses.


Keep in mind that larger memory caches, bitmap pools and disk caches typically provide somewhat better performance, at least up to a point. If you change cache sizes, you should carefully measure performance before and after your changes to make sure the performance/size tradeoffs are reasonable.


#### **Memory Cache**

By default Glide’s memory cache and <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code> respond to <code>[ComponentCallbacks2](https://d.android.com/reference/android/content/ComponentCallbacks2.html?is-external=true)</code> and automatically evict their contents to varying degrees depending on the level the framework provides. As a result, you typically don’t need to try to dynamically monitor or clear your cache or <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code>. However, should the need arise, Glide does provide a couple of manual options.


##### **Permanent size changes**


    To change the amount of RAM available to Glide across your application, see the [Configuration page](https://nickyshe.github.io/Glide-V4/#/Configurations).


##### **Temporary size changes.**

To temporarily allow Glide to use more or less memory in certain parts of your app, you can use <code>[setMemoryCategory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/Glide.html#setMemoryCategory-com.bumptech.glide.MemoryCategory-)</code>:


```
Glide.get(context).setMemoryCategory(MemoryCategory.LOW);
// Or:
Glide.get(context).setMemoryCategory(MemoryCategory.HIGH);
```


Make sure to reset the memory category back when you leave the memory or performance sensitive area of your app:


```
Glide.get(context).setMemoryCategory(MemoryCategory.NORMAL);
```



##### **Clearing memory**

 To simply clear out Glide’s in memory cache and <code>[BitmapPool](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html)</code>, use <code>[clearMemory](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/Glide.html#clearMemory--)</code>:


```
// This method must be called on the main thread.
Glide.get(context).clearMemory();
```

Clearing all memory isn’t particularly efficient and should be avoided whenever possible to avoid jank and increased loading times.


#### **Disk Cache**


Glide provides only limited controls for the disk cache size at run time, but the size and configuration can be changed in an `AppGlideModule`.


##### **Permanent size changes**

To change the amount of sdcard space available to Glide’s disk cache across your application, see the [Configuration page](https://bumptech.github.io/glide/doc/configuration.html#disk-cache).


##### **Clearing the disk cache**

To try to clear out all items in the disk cache, you can use <code>[clearDiskCache](https://bumptech.github.io/glide/javadocs/400/com/bumptech/glide/Glide.html#clearDiskCache--)</code>:


```
new AsyncTask<Void, Void, Void> {
  @Override
  protected Void doInBackground(Void... params) {
    // This method must be called on a background thread.
    Glide.get(applicationContext).clearDiskCache();
    return null;
  }
}
