# Modern, minimal Forge 1.12 template

It's kinda annoying to set up forge 1.12 workspaces in the year of our lord 2023 and for some reason [noone wants to tell you how to do it](https://www.reddit.com/r/feedthebeast/comments/11wykah/). Here's one, using ForgeGradle 5.1 and Gradle 7.6.1, so, modern stuff. It is fairly minimal and unopinionated.

**The license for this template itself is CC0, public domain.** I don't care what you do with it and no credit is required. I'm not going to include a separate `LICENSE` file because it's an extra thing you have to delete when setting up a mod.

## Usage

1. Clone this repo, delete the `.git` folder to disassociate your clone from this repo, then run `git init`.
2. "Poke ForgeGradle": run `./gradlew build` or whatever. ForgeGradle will whirr to life and set up the dev environment for you.
   * Give it time, it takes like six minutes on my PC from an empty ForgeGradle cache. No, it's not stuck.
3. Open this project in IntelliJ or your code editor of choice.
4. (optional) Generate "run configurations" by running the appropriate Gradle task, depending on your IDE:
   * IntelliJ: `genIntelliJRuns` (if you're looking in the "gradle tool window" on the right, it's under the "forgegradle runs" folder)
   * Eclipse: `genEclipseRuns`
   * Visual Studio Code: `genVSCodeRuns`

Next, make it yours:

1. In `gradle.properties`, replace `group`, `name`, and `version`.
2. In `src/main/resources/mcmod.info`, replace `modid`, `name`, `description`, `authorList`, and any other properties you feel like filling out.
3. In `src/main/java/com/example/SampleMod112`, fill out the `MODID`, `NAME`, and `VERSION` strings again, and move the class to your own package (not `com.example`).
   * Or just delete it and make a new `@Mod` class, I'm not your mom.

Finally, delete this file and replace it with your own `README`. Good luck in your modding endeavour.

## Common problems

### `Caused by: java.lang.IllegalStateException: ProjectScopeServices has been closed.` in Gradle setup 

[Goofy ass ForgeGradle bug](https://github.com/MinecraftForge/ForgeGradle/issues/563) that's been around forever. Run `./gradlew stop` in the terminal. Even if you think you didn't start a Gradle daemon, try it anyway - IntelliJ likes to start them on its own when it decides to "sync".

### `Caused by: java.lang.ClassCastException: class jdk.internal.loader.ClassLoaders$AppClassLoader cannot be cast to class java.net.URLClassLoader` when starting the game

The game was launched with Java 9 or later. 1.12 is extremely not compatible with this and explodes instantly.

If you use the `runClient`/`runServer` Gradle tasks, this shouldn't happen, because the buildscript provisions a Java 8 toolchain for you and [ForgeGradle will use it](https://github.com/MinecraftForge/ForgeGradle/blob/0a2c70fc412a4c461db50bc20d77164fd5ff6bfa/src/common/java/net/minecraftforge/gradle/common/util/runs/RunConfigGenerator.java#L266-L267). If you used something like the `genIntellijRuns` task to create an IDE run config, though, you will need to fix the configs yourself.

For IntelliJ: open the run config dropdown and select `Edit Configurations...`, select a problematic configuration, and in the `JDK or JRE` dropdown, select a Java 8 JDK. (The one Gradle provisioned for you is in `(user home directory)/.gradle/jdks`.)

(While "opening the game with the `runClient` task" and "opening the game with an IDE run config" *appear* similar on the surface, they're actually wholly separate systems, which is why Gradle can download a Java 8 JDK and ForgeGradle can generate an IDE run config, but you have to plug them together yourself.)

### `Unable to read a class file correctly` / `There was a problem reading the entry module-info.class in the jar (blah blah)` / `probably a corrupt zip` when starting the game

Something on the classpath is compiled to a classfile format newer than the one used in Java 8. Because it breaks the loading process, classes in that jar will not be visible to mods, but it's otherwise harmless.

To me, this happens to a bunch of `asm-6.2`-related jars, but I'm not sure why those are being added to the classpath in the first place when there's also a perfectly fine copy of ASM 5.2!

### Resources (including `mcmod.info`) are not loading

If you google for this you will find a number of really silly solutions, including "downgrading to super ancient versions of Gradle" or "selecting 'Build and Run with IntelliJ IDEA'". The problem is that Forge 1.12 requires the resources to end up in the same directory as the classes, but by default Gradle puts classes in `./build/classes/java/main` and resources in `./build/resources/main`. The solution is simply telling Gradle to share the resources and classes directory:

```groovy
sourceSets.all { it.output.resourcesDir = it.output.classesDirs.getFiles().iterator().next() }
```

This line is included at the bottom of the sample buildscript, so it shouldn't be an issue.

(Snippets of the form `output.resourcesDir = output.classesDir` are floating around online; those work in Gradle 4, but stopped working after Gradle replaced `classesDir` with the more powerful `classesDirs`.)