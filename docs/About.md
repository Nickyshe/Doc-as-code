
## **About**



* [Introduction](#introduction)
* [Frequently asked questions](#frequently-asked-questions)
    * [How do I depend on an integration library?](#how-do-i-depend-on-an-integration-library)
    * [Why include integration libraries?](#why-include-integration-libraries)
    * [Why isn’t xyz library implemented?](#why-isnt-xyz-library-implemented)
    * [Which version of the integration library should I use?](#which-version-of-the-integration-library-should-i-use)
    * [How do I use a specific version of OkHttp, Volley or other third party library?](#how-do-i-use-a-specific-version-of-okhttp-volley-or-other-third-party-library)
    * [What if I want to depend on an integration library, but I want to register its ModelLoaders or other components myself?](#what-if-i-want-to-depend-on-an-integration-library-but-i-want-to-register-its-modelloaders-or-other-components-myself)

###  **Introduction**

To allow Glide’s functionality to be extended, Glide provides support for integration libraries. Integration libraries range in size and scope, but commonly do things like integrate with networking libraries or add support for new decode types. Placing extensions in separate libraries allows users to select the functionality that’s relevant to them while reducing their APK size and app footprint by excluding functionality that’s not as useful.

This section provides information about each available integration library maintained by Glide.


###  **Frequently asked questions**


####  **How do I depend on an integration library?**


There are three parts to depending on any integration library:

1. Include the corresponding Maven, Gradle, or jar dependency in your build. Integration libraries are optional and not included in either Glide’s jar or maven dependency.
2. Follow the [setup steps for Applications on the configurations page](https://bumptech.github.io/glide/doc/configuration.html#applications), and be sure to include an `AppGlideModule` and a dependency on Glide’s annotation processor.
3. Include a dependency on the integration library itself, if you’re using Maven or Gradle, or add the jar for the integration library to your project. \
Most integration libraries will start to work automatically thanks to Glide’s annotation processor and the <code>[LibraryGlideModule](https://bumptech.github.io/glide/doc/configuration.html#libraries)</code> in each integration library. Specific dependencies and integration steps are listed on each integration libraries page in this section.

#### <strong>Why include integration libraries?</strong>


We strongly believe that your choice of a media library should neither dictate the networking library you use in your app, nor require you to include an additional networking library used only to load images. The integration libraries and Glide’s [ModelLoader](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/model/ModelLoader.html) system allow developers to use a consistent platform for all networking operations across their app


#### **Why isn’t xyz library implemented?**

Because you haven’t written the integration library for it yet! OkHttp and Volley are popular libraries that many developers will find useful, but we certainly don’t mean to exclude any other library. If you’ve written a [ModelLoader](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/load/model/ModelLoader.html) for another library for your app and want to open source it, we’d love to see pull requests to support additional libraries.


#### **Which version of the integration library should I use?**

Starting in Glide v4, the integration libraries use exactly the same version numbers as the main Glide library. The version of the integration libraries you use should exactly match the version of Glide you depend on. Using different versions of Glide and integration libraries can lead to crashes, build errors, or even undefined behavior.


####  **How do I use a specific version of OkHttp, Volley or other third party library?**

The third party libraries that Glide’s integration libraries integrate with may have their own versioning system. Typically the integration libraries depend on third party libraries via transitive Maven/Gradle dependencies. When depending on integration libraries using Maven or Gradle, you can override the version of the third party libraries by explicitly including the third party library at whatever version you use elsewhere in your app.

For example, to override the version of OkHttp used by Glide’s `okhttp-integration` library that might be slightly out of date, you could depend on OkHttp 3.9.1 directly:



```
implementation com.squareup.okhttp3:okhttp:3.9.1
implementation com.github.bumptech.glide:okhttp-integration:4.11.0
```


For more control, you can also exclude the OkHttp from the transitive dependencies of the `okhttp-integration` library:


```
implementation (com.github.bumptech.glide:okhttp-integration:4.11.0) {
  exclude group: com.squareup.okhttp3, module: okhttp
}
```

Or you could simply exclude all transitive dependencies of the `okhttp-integration` library:


```
implementation (com.github.bumptech.glide:okhttp-integration:4.11.0) {
  transitive = false
}
```



#### **What if I want to depend on an integration library, but I want to register its ModelLoaders or other components myself?**

 By default, most integration libraries will have a <code>[LibraryGlideModule](https://bumptech.github.io/glide/doc/configuration.html#libraries)</code> that will automatically register the components of that library when you add a dependency on the project. Sometimes it’s useful to be able to exclude some of those default components, register them in a different order, or customize them. To do so, you’ll need to:

Prevent the default `LibraryGlideModule` from being included by Glide’s annotation processor \
To do so, add an <code>[@Excludes](https://bumptech.github.io/glide/javadocs/431/com/bumptech/glide/annotation/Excludes.html)</code> annotation to your <code>[AppGlideModule](https://bumptech.github.io/glide/doc/configuration.html#conflicts)</code> that references the <code>LibraryGlideModule</code> class you’d like to exclude: \
<code>@Excludes(com.example.unwanted.GlideModule.class)</code>


```
@GlideModule
public final class MyAppGlideModule extends AppGlideModule { }

```



1. For more details, see the [configuration page](https://nickyshe.github.io/Glide-V4/#/Configurations).

Register the components you want in your own `LibraryGlideModule` or `AppGlideModule \
`In your `AppGlideModule`, you can implement the `registerComponents` method to add whatever `ModelLoader`s or other components you’d like: \
`@GlideModule`


```
public class YourAppGlideModule extends AppGlideModule {
  @Override
  public void registerComponents(Context context, Glide glide, Registry registry) {
    registry.append(Photo.class, InputStream.class, new CustomModelLoader.Factory());
  }
}

```



2. For more details, see the [configuration page](https://nickyshe.github.io/Glide-V4/#/Configurations)