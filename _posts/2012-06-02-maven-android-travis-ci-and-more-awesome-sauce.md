---
layout: default
title:  "Maven, Android, Travis-CI and More Awesome Sauce"
---

# Maven, Android, Travis-CI and More Awesome Sauce

##  Background

Recently I started working on an Android application for a side project.  From the beginning, one thing I wanted to accomplish was to setup continuous integration from the get-go.  After looking into it for a bit, I discovered that there were three potential options for me:

* Setup my own CI server using something like [Amazon EC2](http://aws.amazon.com/ec2/) or [Linode](http://www.linode.com/)
* Use [CloudBees](http://wiki.cloudbees.com/bin/view/DEV/Android) (a cloud based Jenkins solution)
* Use [travis-ci](http://travis-ci.org/) (an open-source hosted CI service)

Of those three options, the only one that would allow me to test my application on an actual Android emulator was the first, and since I wanted to spend exactly $0 towards it, the latter two were more appealing.  Since I have traditionally used Jenkins in the past for my Android applications, I decided to go with a combination of [Maven](http://maven.apache.org/), the [maven-android-plugin](http://code.google.com/p/maven-android-plugin/wiki/GettingStarted) and travis-ci for this project.

I don't blog much (as you can see), but after a bit of struggle with getting this building successfully on travis-ci I decided to share my experiences with this.  I have made the [example test project](https://github.com/leviwilson/android-travis-ci-example) available on Github.

#  Creating the project

Using the maven-android-plugin, I created the skeleton application using the example on [android-quickstart-archetype](http://stand.spree.de/wiki_details_maven_archetypes).  Here is the command that I used for my sample project:

{% highlight bash %}
mvn archetype:generate -DarchetypeArtifactId=android-quickstart \
-DarchetypeGroupId=de.akquinet.android.archetypes \
-DarchetypeVersion=1.0.8 \
-DgroupId=com.leviwilson.android \
-DartifactId=android-travis-ci-example \
-Dplatform=10
{% endhighlight %}

The only information that you need to modify from this example would be the `groupId`, `artifactId` and the `platform` that you are building for (note this).  This should be enough to build your project using the `mvn install` target.

## Setting up travis-ci

If we add a vanilla [.travis.yml](https://github.com/leviwilson/android-travis-ci-example/blob/a578cd59e3220ff205af682b121d0fb06f1cdfc2/.travis.yml) file to our application, you'll notice that out of the box [we will get a build failure](http://travis-ci.org/#!/leviwilson/android-travis-ci-example/builds/1511700).

{% highlight yaml %}
language: java
{% endhighlight %}

Not surprisingly it is because the travis-ci build agents do not have any sort of android environment setup on them.   To get around this, we will need to take advantage of the [various hooks in the build lifecycle](http://about.travis-ci.org/docs/user/build-configuration/) that travis-ci provides you to setup an android environment prior to building our application.  In order to do this, we will need to do the following in our `.travis.yml`:

*  download the latest android SDK and unzip it
*  setup the `ANDROID_HOME` environment variable to point to the SDK that we downloaded
*  fix up our `PATH` variable to point to the `tools` and `platform-tools` directory in our SDK
*  tell our android environment to update our local SDK with the target API that we are building our application for

### Choosing the Right Environment
Since travis-ci has a limited amount of space that we can take up and the full Android environment is huge, we will need to tell android to only grab the specific SDK that our application is building for.  To find this information out, you can run the `android list sdk` command from your terminal.  Doing so will give you a list of what is available to update.  Since we are targeting our application for API Level 10, we will want to note the package number (9).

{% highlight basedir %}
Packages available for installation or update: 66
   1- Android SDK Tools, revision 19
   2- Android SDK Platform-tools, revision 11
   3- Documentation for Android SDK, API 15, revision 2
   4- SDK Platform Android 4.0.3, API 15, revision 3
   5- SDK Platform Android 4.0, API 14, revision 3
   6- SDK Platform Android 3.2, API 13, revision 1
   7- SDK Platform Android 3.1, API 12, revision 3
   8- SDK Platform Android 3.0, API 11, revision 2
   9- SDK Platform Android 2.3.3, API 10, revision 2
  10- SDK Platform Android 2.2, API 8, revision 3
  11- SDK Platform Android 2.1, API 7, revision 3
  12- SDK Platform Android 1.6, API 4, revision 3
  13- SDK Platform Android 1.5, API 3, revision 4
  14- Samples for SDK API 15, revision 2
  15- Samples for SDK API 14, revision 2
[...]
{% endhighlight %}

### Platform To Stand On
In addition to choosing the proper platform SDKs that your application builds for, you'll also want to be sure to include both the Android Tools and Android Platform Tools (options 1 and 2) for the SDK that you are targeting.

## Robolectric

For those that haven't at least checked out [Robolectric](http://pivotal.github.com/robolectric/) to test drive your Android applications, do yourself a favor.  I had heard about Robolectric when I first started writing Android applications, but at the time thought it would be more prudent to use the tools available through [android.test](http://developer.android.com/reference/android/test/package-summary.html).  Currently, I support a good mixture between Robolectric at the unit level and android.test at the integration level, but that is a topic for another blog post. 

### Including Robolectric In Your pom.xml
Robolectric has a [good quick start guide](http://pivotal.github.com/robolectric/maven-quick-start.html) to adding it to your Maven project.  The general idea is to have entries for Robolectric and JUnit in your `<dependencies />` entry of your `pom.xml`.  For this project, I've just added a simple test that verifies we get the proper greeting message when the application starts.  Listed below are my `HelloAndroidActivityTest.java` and my `pom.xml`.

{% highlight java %}
package com.leviwilson.android;

import static com.leviwilson.android.R.id;
import static org.junit.Assert.*;
import static org.hamcrest.CoreMatchers.*;

import android.widget.TextView;
import com.xtremelabs.robolectric.RobolectricTestRunner;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(RobolectricTestRunner.class)
public class HelloAndroidActivityTest {

    private HelloAndroidActivity activity;

    @Before
        public void setUp() {
            activity = new HelloAndroidActivity();
            activity.onCreate(null);
        }

    @Test
        public void itProperlyGreetsYou() {
            assertThat(textOf(id.greet_them), equalTo("Hello android-travis-ci-example!"));
        }

    private String textOf(int id) {
        final TextView textView = (TextView)activity.findViewById(id);
        return textView.getText().toString();
    }
}
{% endhighlight %}


{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.leviwilson.android</groupId>
  <artifactId>android-travis-ci-example</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>apk</packaging>
  <name>android-travis-ci-example</name>

  <properties>
    <platform.version> 2.3.3
    </platform.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>com.google.android</groupId>
      <artifactId>android</artifactId>
      <version>${platform.version}</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>org.hamcrest</groupId>
      <artifactId>hamcrest-core</artifactId>
      <version>1.2</version>
      <scope>provided</scope>
    </dependency>


    <dependency>
      <groupId>com.pivotallabs</groupId>
      <artifactId>robolectric</artifactId>
      <version>1.1</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.8.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
        <artifactId>android-maven-plugin</artifactId>
        <version>3.1.1</version>
        <configuration>
          <androidManifestFile>${project.basedir}/AndroidManifest.xml</androidManifestFile>
          <assetsDirectory>${project.basedir}/assets</assetsDirectory>
          <resourceDirectory>${project.basedir}/res</resourceDirectory>
          <nativeLibrariesDirectory>${project.basedir}/src/main/native</nativeLibrariesDirectory>
          <sdk>
            <platform>10</platform>
          </sdk>
          <undeployBeforeDeploy>true</undeployBeforeDeploy>
        </configuration>
        <extensions>true</extensions>
      </plugin>

      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>2.3.2</version>
        <configuration>
          <source>1.6</source>
          <target>1.6</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
{% endhighlight %}

## Putting it all together

Now that you know what API level to include and our Maven project is setup to build our test, the only thing left to do is to perform the steps needed in the `before_install` portion of our [.travis.yml](https://github.com/leviwilson/android-travis-ci-example/blob/5c8e802994075f5be434fae1adabf1406f68828d/.travis.yml).  Here is what our resulting file looks like:

{% highlight yaml %}
language: java
before_install:
  - wget http://dl.google.com/android/android-sdk_r18-linux.tgz
  - tar -zxf android-sdk_r18-linux.tgz
  - export ANDROID_HOME=~/builds/leviwilson/android-travis-ci-example/android-sdk-linux
  - export PATH=${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools
  - android update sdk --filter 1,2,9 --no-ui --force
{% endhighlight %}

And there you have it.  After setting up the appropriate android environment in the travis-ci environment, [we are now green](http://travis-ci.org/#!/leviwilson/android-travis-ci-example/builds/1512189).

## Summary

Though this is a rudimentary example, I still think it is cool to see how powerful travis-ci can be.  I would also like to explore setting up some integration level tests in travis-ci using something similar to how Jenkins uses the XVNC plugin to run an emulator on a server.

{% if page.comments %}
        <div id="disqus_thread"></div>
        <script type="text/javascript">
            /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
            var disqus_shortname = 'leviwilson'; // required: replace example with your forum shortname

            /* * * DON'T EDIT BELOW THIS LINE * * */
            (function() {
                var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
                dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
                (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
            })();
        </script>
        <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
        <a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
        
{% endif %}
