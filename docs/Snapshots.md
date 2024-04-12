
##  **Snapshots**



* [About Snapshots](#about-snapshots)
* [Obtaining Snapshots](#obtaining-snapshots)
    * [Jar](#jar)
    * [Gradle](#gradle)
    * [Maven](#maven)
    * [Fixing your snapshot dependency](#fixing-your-snapshot-dependency)
    * [Building snapshots locally](#building-snapshots-locally)
        * [Installing in the default local Maven repo](#installing-in-a-specific-local-or-remote-maven-repo)
        * [Installing in a specific local or remote Maven repo](#installing-in-a-specific-local-or-remote-maven-repo)

## **About Snapshots**


 For users who don’t want to wait for the next version of Glide and are willing to live on the bleeding edge, we deploy snapshot versions of the library to [Sonatype’s snapshot repo](https://oss.sonatype.org/content/repositories/snapshots/).


After each push to the master branch on GitHub, Glide is built by [travis-ci](https://travis-ci.org/bumptech/glide). If the build succeeds, we automatically deploy the latest version of the library to Sonatype.


 Each integration library will have its own snapshot, as will the main Glide library. If you use a snapshot version of the Glide library you must also use the snapshot versions of any integration libraries you use as well, and vice versa.


##  **Obtaining Snapshots**

 Sonatype’s snapshot repo functions as any other maven repo would, so snapshots are accessible as a jar, in maven, or in gradle.


###  **Jar**


Jars can be downloaded [directly from Sonatype](https://oss.sonatype.org/content/repositories/snapshots/com/github/bumptech/glide/). Double check the date to make sure you’re getting the latest version.


###  **Gradle**


Add the snapshot repo to your list of repositories:



```
repositories {
  jcenter()
  maven {
    name 'glide-snapshot'
    url 'https://oss.sonatype.org/content/repositories/snapshots'
  }
}
```



And then change your dependencies to the snapshot version:


```
dependencies {
  compile 'com.github.bumptech.glide:glide:4.12.0-SNAPSHOT'
  compile 'com.github.bumptech.glide:okhttp-integration:4.12.0-SNAPSHOT'
}
```



### **Maven**


_This is untested and taken from a [Stack Overflow](https://stackoverflow.com/questions/7715321/how-to-download-snapshot-version-from-maven-snapshot-repository) question. Suggestions on improving this section are particularly welcome!_


 Add the following to your `~/.m2/settings.xml`:


```
<profiles>
  <profile>
     <id>allow-snapshots</id>
     <activation><activeByDefault>true</activeByDefault></activation>
     <repositories>
       <repository>
         <id>snapshots-repo</id>
         <url>https://oss.sonatype.org/content/repositories/snapshots</url>
         <releases><enabled>false</enabled></releases>
         <snapshots><enabled>true</enabled></snapshots>
       </repository>
     </repositories>
   </profile>
</profiles>
```



Then change your dependencies to the snapshot version:


```
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>glide</artifactId>
  <version>4.12.0-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>okhttp-integration</artifactId>
  <version>4.12.0-SNAPSHOT</version>
</dependency>
```



### **Fixing your snapshot dependency**

Depending on a `-SNAPSHOT` version of Glide can be risky in production applications because the code Gradle will fetch for that `-SNAPSHOT` version will vary depending on when you first build your project. If you add a snapshot dependency, test on your local machine, and then push that build configuration to a build server, the build server may end up building with a different version of Glide. Snapshot versions are updated after every successful push to GitHub and may change at any time.

To fix your dependency, you can pick a specific version from sonatype and use that instead of depending on `-SNAPSHOT`. For example, in Gradle:


```
dependencies {
  compile 'com.github.bumptech.glide:glide:4.3.0-20171024.022226-26'
  compile 'com.github.bumptech.glide:okhttp-integration:4.3.0-20171024.022226-26'
}
```


Or in Maven (untested):


```
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>glide</artifactId>
  <version>4.3.0-20171024.022226-26</version>
</dependency>
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>okhttp-integration</artifactId>
  <version>4.3.0-20171024.022226-26</version>
</dependency>
```



The version, `4.3.0-20171024.022226-26`, comes from the Sonatype repo. You can pick a specific version by:



1. [Open Sonatype](https://oss.sonatype.org/content/repositories/snapshots/com/github/bumptech/glide/)
2. Click on the package you want. Typically you can just use [glide](https://oss.sonatype.org/content/repositories/snapshots/com/github/bumptech/glide/glide/).
3. Click on the version of Glide that you want the snapshot of. For example, [4.3.0-SNAPSHOT](https://oss.sonatype.org/content/repositories/snapshots/com/github/bumptech/glide/glide/4.3.0-SNAPSHOT/).
4. Copy and paste the version number from any of the listed artifacts. For example if you see `glide-4.3.0-20171024.022211-26-javadoc.jar`, the version number is just `4.3.0-20171024.022211-26`. Usually you will want the most recent artifact available. Check the date modified column to be sure, but typically more recent artifacts are shown at the bottom of the page.

 Although picking a specific snapshot version is a bit more work, it’s typically a safer option if you’re going to depend on the snapshot version of Glide in a prod version of your application or library.


### **Building snapshots locally**

Maven allows you to install artifacts in specific Maven repositories and depend on those artifcats in other projects. Often this is a simple way to test changes in Glide in third party projects. There are two places that you


####  **Installing in the default local Maven repo**

 If you simply want to test that some changes you’ve made in Glide work with your project (or build your project against a specific version or commit of Glide), you can install Glide in the default local maven repository.

 To do so, add the following to the `repositories` section of your `build.gradle` file:



```
repositories {
  mavenLocal()
}
```

Then build glide with `-PLOCAL`:


```
./gradlew uploadArchives --parallel -PLOCAL
```



#### **Installing in a specific local or remote Maven repo**

If you need to specify a specific local or remote Maven repo to install Glide to, you can do so with the following command:


```
./gradlew uploadArchives --stacktrace --info -PSNAPSHOT_REPOSITORY_URL=file://p:\path\to\repo -PRELEASE_REPOSITORY_URL=file://p:\path\to\repo
```


This will create a m2 repository folder that you can consume with Gradle in a project to test your change:


```
repositories {
  // make sure this is declared before glide-snapshot so it's queried first
  maven { name 'glide-local'; url 'p:\\path\\to\\repo' }
}
dependencies {
  // enable this to make sure changes are picked up immediately
  //configurations.compile.resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
  compile 'com.github.bumptech.glide:glide:x.y.z-SNAPSHOT'
}
