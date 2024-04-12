
## **OkHttp3**


### **OkHttp3**

OkHttp is a lower level networking library than Volley. It’s meant more as a replacement for Android’s default networking stack than as a queueing system. OkHttp includes support for SPDY. OkHttp has reasonable performance with Glide and typically generates less garbage than Volley when loading images. OkHttp is a reasonable choice for apps that want a nicer API than the one provided by Android’s HttpUrlConnection or that want to ensure that the networking code they rely on is consistent regardless of which Android OS version their app is installed on. As with all the networking libraries another good reason to use the OkHttp integration library is if you already use OkHttp elsewhere in the app.


#### **How do I include the OkHttp3 integration library?**

First make sure you’ve followed the [setup instructions](https://bumptech.github.io/glide/doc/configuration.html#applications) for Applications.

Then add a Gradle dependency on the OkHttp integration library:


```
implementation "com.github.bumptech.glide:okhttp3-integration:4.11.0"
```


Adding a Gradle dependency on the OkHttp integration library will cause Glide to start automatically using OkHttp to load all images from http and https urls.

For more details on the automatic registration of integration libraries and answers to common questions, see the [About section](https://bumptech.github.io/glide/int/about.html) for integration libraries.
