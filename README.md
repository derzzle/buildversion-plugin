# buildversion-maven-plugin

This is a maven plugin that extracts current build information from git
projects, including: the latest commit hash, timestamp, most recent tag, number
of commits since most recent tag. It also implements a "follow first parent"
flavor of `git describe` (see "About git-describe" below for details).

Similar in intent to
[buildnumber-maven-plugin](http://mojo.codehaus.org/buildnumber-maven-plugin/),
this plugin sets Maven project properties intended to be used by later
phases. You may use this to include build version information on property files,
manifests and generated sources.


## Usage

Simply add `buildversion-plugin` to your pom, executing the `set-properties` goal. Example:


```xml
    ...
     <build>
        <plugins>
          <plugin>
            <groupId>com.code54.mojo</groupId>
            <artifactId>buildversion-plugin</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <executions>
              <execution>
                <goals><goal>set-properties</goal></goals>
              </execution>
            </executions>
          </plugin>
    ...
```

The plugin runs on Maven's `initialize` phase. Any plugin running after that phase will see following properties:

* `build-tag`: Closest repository tag (in commit history following "first parents"). NOTE: Only tags starting with `v` are considered, and the `v` is stripped. Example: `1.2.0-SNAPSHOT` (tag on git: `v1.2.0-SNAPSHOT`).
* `build-tag-delta`: Number of commits since the closest tag until HEAD. Example: `2`
* `build-commit`: Full hash of current commit (HEAD). Example: `c154712b8cea9da812c52f269578a458911f24cc`
* `build-commit-abbrev`: Abbreviated hash of current commit (HEAD). Example: `c154712`
* `build-version`: Full descriptive version of current build. Includes closest tag, tag delta, and abbreviated commit hash. Example: `1.2.0-SNAPSHOT-2-c154712`
* `build-tstamp`: A date and time stamp of the current commit (HEAD). The pattern is configurable. Example: `20120407001823`.


As an example, you may display the value of those properties using the antrun plugin:

```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <executions>
            <execution>
                <id>echo-stuff</id>
                <phase>initialize</phase>
                <goals><goal>run</goal></goals>
                <configuration>
                    <tasks>
                        <echo message="build-version: ${build-version}" />
                        <echo message="build-tag: ${build-tag}" />
                        <echo message="build-tag-delta: ${build-tag-delta}" />
                        <echo message="build-tstamp: ${build-tstamp}" />
                        <echo message="build-commit: ${build-commit}" />
                        <echo message="build-commit-abbrev: ${build-commit-abbrev}" />
                    </tasks>
                </configuration>
            </execution>
        </executions>
    </plugin>
```

Note: `buildversion-plugin` is currently hosted at the `oss.sonatype.org` maven
repository, and may depend on artifacts on the Clojars repository. So, to use this plugin,
you'd need to add these repos to your `settings.xml` or your project pom. Example:


```xml
    <pluginRepositories>
      <pluginRepository>
        <id>sonatype-snapshots</id>
        <url>http://oss.sonatype.org/content/repositories/releases</url>
      </pluginRepository>
      <pluginRepository>
        <id>clojars.org</id>
        <url>http://clojars.org/repo</url>
      </pluginRepository>
    </pluginRepositories>
```


## Configuration

You may configure the format for `build-tstamp` with the `tstampFormat`
configuration option, using the pattern syntax defined by Java's [SimpleDateFormat](http://docs.oracle.com/javase/6/docs/api/java/text/SimpleDateFormat.html). Example:

```xml
	 <plugin>
	   <groupId>com.code54.mojo</groupId>
	   <artifactId>buildversion-plugin</artifactId>
	   <version>1.0.0-SNAPSHOT</version>
	   <executions>
		 <execution>
		   <goals><goal>set-properties</goal></goals>
		<configuration>
			 <tstampFormat>yyyy-MM-dd</tstampFormat>
		   </configuration>
		 </execution>
	   </executions>
	 </plugin>
```


# About git-describe

Before writing this plugin, I used to rely on a simple script which called `git
describe` to obtain a descriptive version number including most recent tag and
commits since such a tag.

Unfortunately, the logic behind `git describe` searches for the closest recent
tag back in history *following all parent commits on merges*, which means, it
may select tags you originally put *on another branch*. So, if you are working on a
development branch and merge back a fix made on a release branch, calling `git
describe` on the development branch may shield a description that includes a tag
you place on the release branch.

Until `git describe` accepts a `--first-parent` argument to prevent this
problem, this plugin implements its own logic, which basically relies on `git
log --first-parent` to traverse history on the current "line of development".

Reference:

 * [Another explanation](http://www.xerxesb.com/2010/git-describe-and-the-tale-of-the-wrong-commits/) of this same issue with `git describe`.
 * [GIT mailing list discussion](http://kerneltrap.org/mailarchive/git/2010/9/21/40071/thread) about `git describe`'s logic and lack of `--first-parent`.
 * Here's a [working patch to add `--first-parent` to `git describe`](https://github.com/gitigit/git/tree/mrb/describe-first-parent)


## Future
Some ideas on my TO-DO list:

 * Find a way to inject the project version from git tags into the project's
  pom. The goal is to get rid of the need to modify the project version inside
  `pom.xml`: just leave a fixed 0.0.0 version and let the plugin infer it from
  git. We tried the obvious: calling project.setVersion(), but some Maven phases
  still "see" the version that is in the `pom.xml` file. Need to research
  further.
 * Expose more configuration options (like, the pattern to match candidate tags)
 * Add a mechanism for the plugin to generate a properties file (or, better, any
   file from a template)
 * Support for other repos? (SVN, git-svn, mercurial)

## FAQ
 * Why don't just use `buildnumber-maven-plugin` ?
   Because it doesn't offer information about tagging, nor commits since tag. See "About git-describe" above for details.
 * Why is implemented in Clojure?
   Because I like it and wanted to experience its Java interoperability. I Thought
   a Maven plugin was a good test-bed given that it's very Java-centric
   (including custom annotations, multiple classloaders, a DI container -Plexus-
   for Java objects, etc).

## Acknowledgments

Thanks to Hugo Duncan for his work on
[clojure-maven](https://github.com/pallet/clojure-maven) and accepting my
`defmojo` addition. He did the heavy-lifting to integrate Clojure with Maven and
Plexus.

## License
Licensed under the [Eclipse Public License](http://www.eclipse.org/legal/epl-v10.html).
Copyright 2012 Fernando Dobladez.
