# Development and Contribution

## Building
1. Run `./gradlew setupDecompWorkspace clean build` command.
1. The built jars are all in `${rootprojectDir}/build/libs` folder.

*NOTICE: The jar file with specific Minecraft version is for the vanilla edition without Forge and Fabric. The json files are launcher profiles which only work when building with our CI server.*

## Running and Testing
For now, CustomSkinLoader is unable to run under self development environment, so it needs to add to another development environment as a library.  
There are the available versions in different environments below:  
|  | Forge | Fabric |
|:-:|:-:|:-:|
| Runtime Environment | forge-1.8-11.14.0.1237 ~ 1.13.2-25.0.22<br/> forge-1.13.2-25.0.42 ~ latest | fabric-loader-0.4.3+build.134 ~ latest<br/> Minecraft 18w43b ~ latest<br/> *fabric-api is not required* |
| Development Environment | ForgeGradle-2.1-SNAPSHOT ~ latest<br/> forge-1.8-11.14.3.1503 ~ 1.12.2-14.23.5.2855<br/> forge-1.13.2-25.0.198 ~ latest | fabric-loom-(?) ~ latest<br/> fabric-loader-0.12.0 ~ latest<br/> Minecraft 18w49a ~ latest<br/> *fabric-api is not required* |

### Preliminary steps for testing local builds
1. Create a new empty minecraft development environment.
1. Add below contents to `build.gradle`:
   ```gradle
   repositories {
       maven {
           url = "file:/${projectDir}/local-repo"
       }
   }
   ```
1. Create these folders in the new project directory, then copy the built jar and source jar into it:
   ```
   Forge:  ./local-repo/mods/CustomSkinLoader_Forge/${version}
   Fabric: ./local-repo/mods/CustomSkinLoader_Fabric/${version}
   ```
   *`${version}` should be repalced with something like `14.13-SNAPSHOT-00` manually*
1. Create a pom file in the same folder:  
   Forge: `CustomSkinLoader_Forge-${version}.pom`
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
       <modelVersion>4.0.0</modelVersion>
       <groupId>customskinloader</groupId>
       <artifactId>CustomSkinLoader_Forge</artifactId>
       <!-- `${version}` should be repalced with something like `14.13-SNAPSHOT-00` manually -->
       <version>${version}</version>
   </project>
   ```
   Fabric: `CustomSkinLoader_Fabric-${version}.pom`
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
       <modelVersion>4.0.0</modelVersion>
       <groupId>customskinloader</groupId>
       <artifactId>CustomSkinLoader_Fabric</artifactId>
       <!-- `${version}` should be repalced with something like `14.13-SNAPSHOT-00` manually -->
       <version>${version}</version>
   </project>
   ```

### For ForgeGradle 2.x ( forge-1.8-11.14.3.1503 ~ forge-1.12.2-14.23.5.2847 )
1. Add below contents to `build.gradle`:
   ```gradle
   dependencies {
       deobfCompile "mods:CustomSkinLoader_Forge:14.13-SNAPSHOT-00"
   }

   minecraft {
       clientRunArgs += ["--tweakClass", "customskinloader.forge.ForgeDevTweaker", "--username", "<Your username>"]
   }
   ```
1. Run `./gradlew setupDecompWorkspace eclipse` command.
1. Due to the run configurations which generated by ForgeGradle 2 has not been compatible with IntelliJ IDEA 2020 and above, we recommend to install [Eclipser](https://plugins.jetbrains.com/plugin/7153-eclipser) plugin. It can convert Eclipse run configurations to IDEA. (Right click to `*.launch` file in project folder, then select `Convert with Eclipser`, then a new IDEA run configuration will be created.)
1. For IDEA, open `Run/Debug Configurations` dialog, change classpath of module to the name contains `.main`.
1. Add `--tweakClass customskinloader.forge.ForgeDevTweaker --username <Your username>` to CLI arguments in `Run/Debug Configurations` dialog.
1. Then you can debug the mod in IDE or through `./gradlew runClient` command.

### For ForgeGradle 3.x ~ 5.x ( forge-1.12.2-14.23.5.2851 ~ latest )
1. Add below contents to `build.gradle`:
   ```gradle
   dependencies {
       implementation fg.deobf("mods:CustomSkinLoader_Forge:14.13-SNAPSHOT-00")
   }

   minecraft {
       runs {
           client {
               args += ["--username", "<Your username>"]
               args += ["--tweakClass", "customskinloader.forge.ForgeDevTweaker"] // Only required for MinecraftForge 1.12.2
           }
       }
   }
   ```
1. Setup the development environment and run the game as usual.

### For fabric-loom ( fabric-loader-0.12.0 ~ latest )
1. Add below contents to `build.gradle`:
   ```gradle
   dependencies {
       modImplementation "mods:CustomSkinLoader_Fabric:14.13-SNAPSHOT-00"
   }

   tasks.runClient {
	   args += ["--username", "<Your username>"]
   }
   ```
1. Add `--username <Your username>` to CLI arguments in `Run/Debug Configurations` dialog.
1. Run the game in IDE or through `./gradlew runClient` command..

## Depend on release builds
1. Check the latest version in https://littlesk.in/csl-latest .
1. Add below contents to `build.gradle`:
   ```gradle
   // Before Gradle 5.x
   repositories {
       ivy {
           url = "https://csl.littleservice.cn/"
           layout "pattern", {
               artifact "[organisation]/[artifact]-[revision](-[classifier])(.[ext])"
           }
       }
   }
   
   // After Gradle 6.x
   repositories {
       ivy {
           url = "https://csl.littleservice.cn/"
           metadataSources {
               artifact()
           }
           patternLayout {
               artifact "[organisation]/[artifact]-[revision](-[classifier])(.[ext])"
           }
       }
   }
   ```
1. Follow the same steps in **Running and Testing**.

## Developing
- CustomSkinLoader is based on forge-1.12.2-14.23.5.2768 currently, including Fabric edition. We use custom reobfuscation mappings such as [Fabric.tsrg](Fabric/Fabric.tsrg) and [mixin.tsrg](Fabric/mixin.tsrg) to generate different editions jar and Mixin reference jsons.
- The source codes are all in `${projectDir}/sources` and resource files are in `${projectDir}/resources` instead of `src/main/java` and `src/main/resources`.
- Do not add other required mods.