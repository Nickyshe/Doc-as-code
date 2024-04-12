## **Samples**


####  **Sample Apps**


Glide includes a number of sample apps in the [samples/](https://github.com/bumptech/glide/tree/master/samples) directory that demonstrate how to load images with Glide in a variety of contexts.


The sample apps are all built with gradle, so most relevant code will be under sample_app_name/src/main.


Sample apps can be built by:



1. Clone the [Glide repo](https://github.com/bumptech/glide) from GitHub.
2. Run: `./gradlew :samples:&lt;sample_name>:build`

    If you want to automatically install and open the sample app you can use:


    ```
    ./gradlew :samples:<sample_name>:run
    ```



#####  **Flickr**


The Flickr app allows users to search for images matching keywords using Flickr’s public API, and then downloads the first few hundred hits and displays them in a couple of different sizes.

* [Source Code](https://github.com/bumptech/glide/tree/master/samples/flickr)
* Build with: `./gradlew :samples:flickr:run`

##### **Gallery**


The Gallery app displays images and video stills from on the device in a horizontally scrolling RecyclerView.

* [Source Code](https://github.com/bumptech/glide/tree/master/samples/gallery)
* Build with: `./gradlew :samples:gallery:run`

##### **Giphy**

The Giphy app downloas metadata for and popular animated GIFs using [Giphy’s public API](https://api.giphy.com/) and displays them in a vertical list.

* [Source Code](https://github.com/bumptech/glide/tree/master/samples/giphy)
* Build with: `./gradlew :samples:giphy:run`

#####  **SVG**

 The SVG sample app demonstrates how you can use Glide’s flexible decoding pipeline to decode custom resource types. The SVG app loads SVG data from resources and over the network, and uses a custom decoder and drawable to display the SVG with Glide.

* [Source Code](https://github.com/bumptech/glide/tree/master/samples/svg)
* Build with: `./gradlew :samples:svg:run`

#####  **Imgur**

The Imgur sample app retrieves a list of animated and non-animated images from Imgur and displays them in a vertically scrolling list.

* [Source Code](https://github.com/bumptech/glide/tree/master/samples/imgur)
* Build with: `./gradlew :samples:imgur:run`

####  **Open Source Apps**


#####  **Google I/O**

The 2014 Google I/O app used Glide to display images in a variety of scenarios. The Google I/O app is available on GitHub. In addition, the I/O team wrote a number of useful blog posts, including one on [image loading](https://github.com/google/iosched/blob/master/doc/IMAGES.md), which may be useful. For more posts, see their [Readme.md](https://github.com/google/iosched/blob/master/README.md#how-to-work-with-the-source).

* [Source Code](https://github.com/google/iosched)