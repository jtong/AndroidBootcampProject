###How I Got BDD-TDD Working In Android Studio 0.4.3

My goal was to be able to drive a [BDD-TDD cycle](http://boostagile.com/test-driven-development-and-agile/) using Android instrument tests with Espresso for the outer BDD cycle, and JUnit with Robolectric for the inner TDD cycle.

This turned out to be a non-trivial exercise, as JUnit is not officially supported as of AS 0.4.3


1. Installed the standalone Espresso library as per [the instructions](https://code.google.com/p/android-test-kit/wiki/Espresso). (The separate library with dependencies did not work.)
2. Wrote some [behaviour tests](https://github.com/macosgrove/AndroidBootcampProject/commit/ff41e46f18da9ab904607f62a766368459b78db2). These were easy to run within AS and failed in the expected way (as the code is not yet implemented).
3. Following advice from novoda's [gradle-android-test-plugin](https://github.com/novoda/gradle-android-test-plugin), created a [separate module for unit tests and added the plugin to gradle for that module](https://github.com/macosgrove/AndroidBootcampProject/commit/816cc9f7dc56a3d3d09b040891992d336f6bc277). Now, running the tests from the command line with 
```gradle check --debug``` 
results in this error: 
```java.lang.NoClassDefFoundError: org/gradle/api/artifacts/result/ResolvedModuleVersionResult```
4. Downgraded gradle to v1.9 [using homebrew](http://stackoverflow.com/questions/3987683/homebrew-install-specific-version-of-formula). Now running the tests from the command line with 
```gradle check```
sometimes works, and sometimes reports a success without compiling or running the tests.
```gradle check --info```
or 
```gradle check --debug```
seem to work consistently. Note that the first run takes a very long time as Robolectric downloads.  
At this stage, running the tests within AS results in
```!!! JUnit version 3.8 or later expected```
5. Following the advice [here](http://kostyay.name/android-studio-robolectric-gradle-getting-work/), obtained the classpath from the console line above the previous error message, modified it to move JUnit 4.11 to the head of the path and add my test-debug output directory, and added it to the VM Options in the run configuration in AS. Hurray, tests now run in AS! HOWEVER, the test source is not compiled within AS. If you clean the project and re-run the test in AS, you get this error:
```Class not found: "com.thoughtworks.androidbootcamp.controller.HelloAndroidTest"```
6. Created a new run configuration for gradle. Set task to compileTestDebugJava. Added it to the 'before launch' part of the JUnit test run configuration. At this stage, the test and compile run in parallel and the compile finishes after the test finishes so you still get the class not found exception if the project has been cleaned.
7. Moved the compile run configuration to before the Make in the 'before launch'. Now the test works!
8. Made the same changes to the Default JUnit run configuration so future tests would run correctly.
9. At this stage simple tests work but Robolectric can't create any views as it doesn't know where to find the resources.
Created Robolectric configuration file org.robolectric.Config.properties under src/test/resources and added the folder to the classpath in the run configurations.
10. Robolectric still can't find the Android resources in ```/Users/macosgrove/AndroidStudioProjects/AndroidBootcampProject/AndroidBootcamp/build/exploded-bundles/ComAndroidSupportAppcompatV71900.aar/res```  
Result is this:  
```java.lang.RuntimeException: huh? can't find parent for StyleData{name='AppTheme', parent='Theme_AppCompat_Light_DarkActionBar'}```
11.  Upgraded to Robolectric 2.3-SNAPSHOT. Installed Maven using ```brew install maven```, set ANDROID_HOME to ```/Applications/Android Studio.app/sdk``` and followed instructions at [Robolectric's github](https://github.com/robolectric/robolectric) to install AppCompat V4. Added AppCompat V4 dependency to testCompile in the test build.gradle.
12. Still giving the ```huh?``` error. Next tried adding test-project.properties and project.properties in same folder as AndroidManifest.xml.
Project.properties:  
```
target=android-18
android.library.reference.1=../AndroidBootcamp/build/exploded-bundles/ComAndroidSupportAppcompatV71901.aar/res
```
Test.properties:
```
android.library.reference.1=../AndroidBootcamp/build/exploded-bundles/ComAndroidSupportAppcompatV71901.aar/res
```
Tried various values for the path. Nothing changed the outcome.  
13. Removed the use of support classes and raised the minimum target sdk version to 14. This now works.
