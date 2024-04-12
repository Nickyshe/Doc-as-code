## **Volley**

Volley is a Java networking queue that supports request queuing and prioritization. Volley isn’t particularly efficient for loading images because it copies all data it receives into byte arrays. Although Volley attempts to re-use these byte arrays, it’s recycling rate is relatively poor for moderate or large images. As a result, Volley can tend to cause a fair amount of memory churn when used with Glide to load images. Volley is still an appropriate choice if already in use by an app because it allows prioritization across both image loading and metadata RPCs. Volley can also be somewhat more robust in poor network than Glide’s default networking library because it includes support for retries.

Typically you will either want to disable Volley’s disk cache or Glide’s disk cache when using this integration library. If you don’t do so, identical data may be kept in both Glide’s and Volley’s disk caches simultaneously.


#### **How do I include the Volley integration library?**

First make sure you’ve followed the [setup instructions](https://nickyshe.github.io/Glide-V4/#/Configurations#applications) for Applications.

Then add a Gradle dependency on the Volley integration library:


```
implementation "com.github.bumptech.glide:volley-integration:4.14.2"
```


Adding a Gradle dependency on the Volley integration library will cause Glide to start automatically using Volley to load images for all http and https urls.

For more details on the automatic registration of integration libraries and answers to common questions, see the [About section](https://nickyshe.github.io/Glide-V4/#/About) for integration libraries.
