
## **Cronet**


### **Cronet**

Cronet is the Chromium network stack made available to Android apps as a library used by many Google apps, including Google Photos. Cronet works very well with Glide and is highly recommended. Cronet is similar to OkHttp in that they’re both lower level libraries that typically replace the default networking stack on the device. [See the Android docs for details](https://bumptech.github.io/glide/int/about.html).

Cronet is a good default choice with Glide and will provide better performance than Glide’s default integration with the standard Android networking stack. As with the other networking libraries, Cronet is a good choice if you’re using it elsewhere in your app. However if you’re already using another networking stack like OkHttp, you may prefer to to use OkHttp with Glide over including another networking library.


#### **How do I include the Cronet integration library?**

First make sure you’ve followed the [setup instructions][3] for Applications.

Then add a Gradle dependency on the Cronet integration library:


```
implementation "com.github.bumptech.glide:cronet-integration:4.14.2"
```


Adding a Gradle dependency on the Cronet integration library will cause Glide to start automatically using Cronet to load all images from http and https urls.

For more details on the automatic registration of integration libraries and answers to common questions, see the [About section](https://nickyshe.github.io/Glide-V4/#/About) for integration libraries.
