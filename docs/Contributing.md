
## **Contributing**



* [Source](#source)
    * [Contribution workflow.](#contribution-workflow)
    * [Building the project.](#building-the-project)
    * [Testing changes](#testing-changes)
        * [Tests](#tests)
            * [Unit tests](#unit-tests)
            * [Annotation Processor tests](#annotation-processor-tests)
            * [Instrumentation tests.](#instrumentation-tests)
        * [Sample projects](#sample-projects)
    * [Code Style](#code-style)
* [Documentation](#documentation)
    * [Via GitHub](#via-github)
    * [Via the Git Branch](#via-the-git-branch)
        * [Modifying existing pages](#modifying-existing-pages)
        * [Adding a new page](#adding-a-new-page)
        * [Viewing your local changes](#viewing-your-local-changes)


### **Source**

Contributions to Glide’s source are welcome!


#### **Contribution workflow.**

To make contributions to Glide’s source:



1. [Fork](https://help.github.com/articles/fork-a-repo/) the [Glide repo](https://github.com/bumptech/glide) on GitHub.


2. [Clone](https://help.github.com/articles/cloning-a-repository/) the [Glide repo](https://github.com/bumptech/glide) from GitHub on to your computer: \

```
git clone https://github.com/&lt;your_username>/glide.git

cd glide

```

3. Open Glide in Android Studio (directions are for Android Studio 3.0+)
    1. Open Android Studio
    2. Click on ‘Import project’
    3. Browse to the directory where you cloned Glide earlier.
    4. Click on ‘settings.gradle’
    5. Click on ‘Open’
4. Make your contributions.
5. Build the project (see below) and make sure the build passes

6. Commit your changes:


```
git add .
git commit -m "Describe your change here."

```




7. Push your changes to your fork of Glide:

`git push origin master`

8. Open your fork of Glide on GitHub (`https://github.com/&lt;your_username>/glide`)
9. Open a [pull request](https://help.github.com/articles/creating-a-pull-request/) from your fork to the main Glide repo on GitHub.


#### **Building the project.**

To build the project, you typically need to run a single gradle command from the project root directory:


```
./gradlew build
```


Glide currently requires a Java version &lt;= 16 to build.


#### **Testing changes**


##### **Tests**

Glide has two types of tests, unit tests that run on your local machine, and instrumentation tests that run on an emulator or device.


###### **Unit tests**

Glide’s unit tests are run as part of Glide’s build, so you can just use:


```
./gradlew build
```


For a faster development cycle, you can also just run the unit tests for the main library with:


```
./gradlew :library:testDebugUnitTest
```



###### **Annotation Processor tests**

To test annotation processor changes run:


```
./gradlew :annotation:compiler:test:test
```


If you changed the output and the regression tests are failing, you can re-generate the test files by running:


```
./gradlew :annotation:compiler:test:regenerateTestResources
```


If you do run `regenerateTestResources`, double check and make sure that the resulting files are sane and that you only see the changes you expected.


###### **Instrumentation tests.**

To run Glide’s instrumentation tests, you need to either plug in a real device, or add an emulator using Android Studio. It’s now quite easy to add an emulator in Android Studio and x86 emulators are quite fast to boot and run. As a result, I’d generally recommend running Glide’s instrumentation tests on an emulator.

To run Glide’s instrumentation tests:



1. [Setup an emulator in Android Studio](https://developer.android.com/studio/run/managing-avds.html#createavd) (I usually use x86 and API 26)
2. Run: \
`./gradlew :instrumentation:connectedDebugAndroidTest`


##### **Sample projects**

Glide’s tests are not completely comprehensive. To verify your changes work and do not negatively affect performance, it’s also a good idea to try running one or more of Glide’s sample projects.

Glide’s sample projects are located in samples/. Sample projects can be built and installed onto a device or emulator using gradle:


```
./gradlew :samples:<sample_name>:run
```


For example, to run the Flickr demo:


```
./gradlew :samples:flickr:run
```



#### **Code Style**

Glide uses [Google’s Java style guide](https://google.github.io/styleguide/javaguide.html).

To configure Android Studio to use Google’s style automatically, use the following steps:



1. Open [https://raw.githubusercontent.com/google/styleguide/gh-pages/intellij-java-google-style.xml](https://raw.githubusercontent.com/google/styleguide/gh-pages/intellij-java-google-style.xml)
2. Save intellij-java-google-style.xml as a file to your computer
3. Open Android Studio
4. Open Preferences…
5. Open Editor > Code Style
6. Next to ‘Schema’ click ‘Manage’
7. Click Import…
8. Highlight ‘Intellij IDEA code style XML’ and click ‘Ok’
9. Browse to the location you downloaded intellij-java-google-style.xml in step 2, select the file, and click ‘Ok’
10. Click ‘Ok’ (you can optionally update the name from GoogleStyle here)
11. In the Code Style Schemes dialog, highlight the style you just created and click ‘Copy to project’
12. Click ok to exit preferences.

To reformat a file after adding the style guide to Android Studio, open the ‘Code’ menu, then click ‘Reformat Code’.

All new code should follow the given style guide and there’s some automated enforcement of the style guide in Glide’s test suite. Pull requests to fix style issues in Glide’s existing code are welcome as well. However, it’s generally best to keep changes that fix style guide issues in existing code and changes that add new code separate. Two pull requests are completely fine if you want to fix some style issues and contribute some new functionality or bug fixes.

When in doubt, send us a single pull request and we will work with you.


### **Documentation**


#### **Via GitHub**

To make simple contributions to existing pages on this documentation site, use GitHub’s UI for editing markdown files. To do so:



1. Sign in to GitHub
2. Open the page you want to edit on https://bumptech.github.io/glide
3. Click ‘Edit this page’ in the page header to open the md file for the page in GitHub’s UI
4. Click the pencil icon to open GitHub’s editor. This will only be clickable if you’ve signed in to GitHub
5. Make your edits
6. Scroll to the bottom of GitHub’s UI, and click the buttons to create a pull request.


#### **Via the Git Branch**

For more involved changes where you want to add or remove a page, change the order in which their displayed, add a new section or perform any other more advanced task, you’ll need to clone the git branch where the documentation site is stored. To do so:



1. [Fork](https://help.github.com/articles/fork-a-repo/) the [Glide repo](https://github.com/bumptech/glide) on GitHub.

2. [Clone](https://help.github.com/articles/cloning-a-repository/) the [Glide repo](https://github.com/bumptech/glide) from GitHub on to your computer: 


```
git clone https://github.com/&lt;your_username>/glide.git

cd glide

```



3. Checkout the gh-pages branch: 
<code>git checkout <strong>-t</strong> origin/gh-pages</code>




4. Make your contributions.

5. Commit your changes: 



```
git add .
git commit -m "Describe your change here."

```



6. Push your changes to your fork of Glide: 
`git push origin gh-pages `


7. Open your fork of Glide on GitHub (`https://github.com/&lt;your_username>/glide`)
8. Open a [pull request](https://help.github.com/articles/creating-a-pull-request/) from your fork to the main Glide repo on GitHub with the `gh-pages` branch.


##### **Modifying existing pages**

The pages you see on the docs page are located in the `_pages` folder and can be modified in place.


##### **Adding a new page**

New pages can be added using `./bin/jekyll-page &lt;page_name> &lt;category>`. `&lt;page_name>` is the title of the page, `&lt;category>` matches one of the sections in the left hand nav. Typically `&lt;category>` should be `doc` so that the page is placed under the `Documentation` section.

When adding a new page, make sure you add `disqus: 1` and `order: &lt;n>` to the header. The number you give to order is used to order the pages within the subsection, with 0 being the first page. To just add the page at the end (a reasonable default), find the order value for the last page in the section and use that value + 1 for your new page.

The final header will look something like this:


```
---
layout: page
title: "Targets"
category: doc
date: 2015-05-26 07:03:23
order: 6
disqus: 1
---
```



##### **Viewing your local changes**

To view your local changes, you will need to install jekyll and the gems used by the docs page:


```
sudo gem install jekyll redcarpet pygments.rb
```


Then you can run jekyll locally:


```
jekyll serve --watch
```


Finally you can view a version of the site with your local changes: `http://127.0.0.1:4000/glide/`. The exact address will be printed out by jekyll.
