Daikon includes a version of the junit-platform-console-standalone jar file to
support progams using JUnit5. Our version is based on junit-platform-console-standalone-1.9.3.jar.
Unfortunately for Daikon users, portions of the official releases contain Java 5 bytecodes.
When a program that uses JUnit5 is processed by DynComp it produces lots of warnings
about the use of old byte codes. More importantly, the Java 5 bytecodes generated for
try finally statements confuse the BCEL bytecode verifier and it reports a fatal error.

This version has been changed to have -target 1.8.  It also has a few type annotations for
the Checker Framework.

Note that the junit-platform-console-standalone jar file contains a shadow copy of the
picocli jar file which has the same problem. Thus you must build a modified version
of picocli first. This is simply done by recompiling.

You must use JDK 11 to build picocli.

To build picocli
----------------
This step builds picocli and copies it to your local maven repository.
It will update some files in the docs directory - these can be ignored.

If you have not already done so:
  git clone git@github.com:remkop/picocli.git
  cd picocli
No files need to be modified.
  git co v4.7.6
  ./gradlew publishToMavenLocal |& tee b2.log


Now you are ready to build JUint 5.
You must JDK 17 for this process.

To build this project
---------------------

If you have not already done so:
  git clone git@github.com:typetools/junit5.git
  cd junit5

You now need to modify three files:
  junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/descriptor/JupiterEngineDescriptor.java

The change identifies this version of the jar as being made for Daikon.
@@ -37,3 +37,3 @@ public class JupiterEngineDescriptor extends EngineDescriptor implements Node<Ju
        public JupiterEngineDescriptor(UniqueId uniqueId, JupiterConfiguration configuration) {
-               super(uniqueId, "JUnit Jupiter");
+               super(uniqueId, "JUnit Jupiter Daikon");
                this.configuration = configuration;

The next two changes ensure that the build uses the picocli you have just built.

gradle/libs.versions.toml
approx line 46:
-picocli = { module = "info.picocli:picocli", version = "4.6.2" }
+picocli = { module = "info.picocli:picocli", version = "4.7.6" }

settings.gradle.kts
approx line 29 add the mavenLocal repository:
        repositories {
+               mavenLocal()
                mavenCentral()

After you have made these edits, build the jar file with:

```
./gradlew junit-platform-console-standalone:clean
./gradlew junit-platform-console-standalone:build
```
The result of the build will be:
 ./junit-platform-console-standalone/build/libs/junit-platform-console-standalone-1.9.0.jar

Note: Do not check in the change to JupiterEngineDescriptor.java; this will cause the github
CI tests to fail.

Test it in a branch of Daikon:
 * copy the jar file to the `daikon/java/lib` directory adding '-Daikon' to the file name prior to '.jar'.
 * remove the old jar file (if the version number has changed)
 * in a Daikon branch run: make compile daikon.jar dyncomp-jdk
 * run:  make MPARG=-j1 -C tests clean diffs
   If there are any errors, then fix the bugs in junit and/or Daikon.
 * push your Daikon branch, and ensure that the CI tests pass
 * merge your branch into master

