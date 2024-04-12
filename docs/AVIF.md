
## **AVIF**


### **AVIF**

[AVIF](https://developer.android.com/about/versions/12/features#avif) is a royalty free image format developed by the [Alliance for Open Media](https://aomedia.org/). It is based on the [AV1](https://en.wikipedia.org/wiki/AV1) video codec. AVIF Integration for Glide uses a bundled [AVIF decoder](https://github.com/AOMediaCodec/libavif) to decode and render AVIF images.

The AVIF Integration supports still AVIF images of all bitdepths (8, 10 or 12) and all YUV configurations (420, 422, 444 or monochrome). It does not support animated AVIF images.


#### **How do I include the AVIF integration library?**

First make sure you’ve followed the [setup instructions](https://nickyshe.github.io/Glide-V4/#/Configurations#applications) for Applications.

Then add a Gradle dependency on the AVIF integration library:


```
implementation "com.github.bumptech.glide:avif-integration:4.14.2"
```


Adding a Gradle dependency on the AVIF integration library will cause Glide to start automatically using the bundled AVIF decoder for decoding and rendering AVIF images.

For more details on the automatic registration of integration libraries and answers to common questions, see the [About section](https://nickyshe.github.io/Glide-V4/#/About) for integration libraries.
