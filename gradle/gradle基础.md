# 核心概念



## 核心概念



![image-20240601170103926](/Users/yinxl/Library/Application Support/typora-user-images/image-20240601170103926.png)



Gradle 根据构建脚本中的信息自动构建，测试和部署软件

### 项目（Projects）

Gradle项目是可以构建的软件片段，比如一个应用程序或一个库。

-   **单项目构建（Single project builds）**：包括一个被称为根项目（root project）的单个项目。
-   **多项目构建（Multi-project builds）**：包括一个根项目和任意数量的子项目（subprojects）。

### 构建脚本（Build Scripts）

构建脚本告诉Gradle需要采取哪些步骤来构建项目。

-   每个项目可以包含一个或多个构建脚本。
-   构建脚本一般是以`build.gradle`文件的形式存在。

### 依赖管理（Dependency Management）

依赖管理是一种自动化技术，用于声明和解析项目所需的外部资源。

-   每个项目通常包含许多外部依赖项，Gradle会在构建过程中解析这些依赖项。
-   依赖项可以是库、框架或其他项目资源。

### 任务（Tasks）

任务是基本的工作单位，比如编译代码或运行测试。

-   每个项目包含一个或多个在构建脚本或插件中定义的任务。
-   任务可以通过命令行执行，比如`gradle build`。

### 插件（Plugins）

插件用于扩展Gradle的功能，并可以选择性地为项目贡献任务。

-   插件可以用来添加新的任务、配置项目或集成其他工具。
-   例如，Java插件可以添加编译Java代码的任务。



### Gradle 项目结构



```
project
├── gradle                              	1
│   ├── libs.versions.toml              	2
│   └── wrapper		
│       ├── gradle-wrapper.jar   					
│       └── gradle-wrapper.properties			
├── gradlew                             3
├── gradlew.bat                         3
├── settings.gradle(.kts)               4
├── subproject-a
│   ├── build.gradle(.kts)              5
│   └── src                             6
└── subproject-b
    ├── build.gradle(.kts)              5
    └── src                             6
```




| 序号 | 介绍                                                         |
| ---- | ------------------------------------------------------------ |
| 1    | Gradle 目录用于存储包装器文件等                              |
| 2    | 用于依赖管理的 Gradle 版本目录，idea自动创建的项目中没看到这个 |
| 3    | Gradle 包装器脚本                                            |
| 4    | Gradle 设置文件定义根项目名称和子项目，项目结构              |
| 5    | 两个子项目的 Gradle 构建脚本 -`subproject-a`以及`subproject-b`<br />单项目的话，build.gradle 文件会放在根目录底下 |
| 6    | 项目的源代码和/或附加文件                                    |



### settings.gradle文件示例

在多项目构建中，`settings.gradle`文件定义了项目的结构和子项目。

```
groovy
复制代码
rootProject.name = 'my-multi-project'

include 'projectA'
include 'projectB'
```

### build.gradle文件示例

根项目的`build.gradle`文件通常定义全局的构建逻辑和依赖。

```
groovy
复制代码
plugins {
    id 'java'
}

subprojects {
    apply plugin: 'java'

    repositories {
        mavenCentral()
    }

    dependencies {
        testImplementation 'junit:junit:4.13.2'
    }
}
```

子项目的`build.gradle`文件可以定义特定于该子项目的构建逻辑和依赖。

`projectA/build.gradle`示例：

```
groovy
复制代码
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:2.5.4'
}
```

`projectB/build.gradle`示例：

```
groovy
复制代码
dependencies {
    implementation 'org.apache.commons:commons-lang3:3.12.0'
}
```

### Gradle Wrapper

Gradle Wrapper是一个非常有用的工具，确保所有开发人员使用相同版本的Gradle构建项目。

`gradle-wrapper.properties`示例：

```
properties
复制代码
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.0-bin.zip
```

GRADLE_USER_HOME 表示用

使用Gradle Wrapper命令：

```
sh
复制代码
./gradlew build
```

### 总结

Gradle 8及以后的版本提供了灵活且强大的构建工具，通过合理的项目结构和配置，可以轻松管理单项目和多项目构建。掌握这些基础知识，可以帮助你高效地使用Gradle进行项目的构建和管理。



## Gradle 包装器基础知识



**执行任何 Gradle 构建的推荐方法**是使用 Gradle Wrapper。

![gradle 基础 2](https://docs.gradle.org/8.5/userguide/img/gradle-basic-2.png)

### 流程图介绍

在 Gradle 的构建流程图中，`Cache`、`Build Flow` 和 `Dependency Manager` 是构建过程中重要的组成部分，分别起着不同的作用：

1.  **Cache（缓存）**：

    -   **作用**：缓存用于存储构建过程中生成的中间结果和最终结果，以避免重复计算和构建，加速后续的构建过程。

    -   类型

        ：

        -   **构建缓存**：缓存任务的输出结果。例如，编译后的类文件、测试结果等。如果相同的任务再次执行且输入未变化，Gradle 可以直接使用缓存的输出结果，而无需重新执行任务。
        -   **依赖缓存**：缓存项目依赖的外部库和插件，从远程仓库下载的依赖会存储在本地缓存中，避免每次构建时重新下载。

    -   **配置**：可以通过配置文件（如 `gradle.properties`）设置缓存策略和路径。

2.  **Build Flow（构建流程）**：

    -   **作用**：描述整个构建过程的各个阶段和步骤，包括初始化、配置、执行等阶段。

    -   阶段

        ：

        -   **初始化阶段**：确定要构建的项目和子项目，设置项目的基本属性。
        -   **配置阶段**：解析和评估所有项目的构建脚本，创建和配置任务。
        -   **执行阶段**：根据任务的依赖关系图，按顺序执行任务。

    -   **任务**：构建流程中的每个任务执行具体的操作，如编译代码、运行测试、打包等。任务之间可以有依赖关系，形成任务依赖图。

3.  **Dependency Manager（依赖管理器）**：

    -   **作用**：管理项目的依赖，包括声明、解析和下载依赖，以及处理依赖冲突。

    -   依赖声明

        ：在 

        ```
        build.gradle
        ```

         文件中，通过 

        ```
        dependencies
        ```

         块声明项目所需的外部库、模块和插件。例如：

        ```
        groovy
        复制代码
        dependencies {
            implementation 'org.springframework.boot:spring-boot-starter-web'
            testImplementation 'junit:junit:4.13.2'
        }
        ```

    -   **依赖解析**：Gradle 解析依赖声明，从本地缓存或远程仓库（如 Maven Central、JCenter 等）下载依赖。

    -   **依赖配置**：可以配置不同的依赖范围（如 `implementation`、`testImplementation`、`compileOnly` 等），指定依赖的作用范围和生命周期。



-   **缓存**与**构建流程**：在构建流程中，Gradle 会利用缓存来加速任务的执行。通过检查缓存，Gradle 可以跳过那些输出未变化的任务，直接使用缓存的结果。
-   **构建流程**与**依赖管理器**：在配置阶段，Gradle 依赖管理器会解析 `build.gradle` 文件中的依赖声明，下载和管理依赖。在执行阶段，Gradle 会确保所需的依赖已经解析和下载完毕，并将其添加到构建路径中。
-   **缓存**与**依赖管理器**：依赖管理器下载的依赖会存储在本地缓存中，避免每次构建时重新下载。



### 包装器介绍

*Wrapper*脚本调用已声明的 Gradle 版本，并在必要时预先下载它。



![wrapper workflow](https://docs.gradle.org/8.5/userguide/img/wrapper-workflow.png)



`gradle-wrapper.properties` 文件是 Gradle Wrapper 的配置文件，用于配置 Gradle Wrapper 在执行构建时所使用的 Gradle 版本和其他相关属性。该文件通常位于项目的根目录下的 `gradle/wrapper` 目录中。

下面是一个典型的 `gradle-wrapper.properties` 文件示例及其用法：

```properties
# 补充：idea中springboot项目，gradle的默认配置
# 指定gradle的分发文件的存储的基础目录
distributionBase=GRADLE_USER_HOME
# gradle分发文件存储的相对路径，即分发文件存储在 ${distributionBase}/${distributionPath}路径下
distributionPath=wrapper/dists
# 指定 Gradle 分发文件的下载地址
distributionUrl=https\://services.gradle.org/distributions/gradle-7.5.1-bin.zip
# 指定 Gradle 分发文件的存储基础目录为 Gradle 用户主目录
zipStoreBase=GRADLE_USER_HOME
# 指定 Gradle 分发文件存储的相对路径为 wrapper/dists
zipStorePath=wrapper/dists
```

**这里需要理解一下什么是分发文件？**

分发文件实际上我感觉应该翻译成**gradle的发行版文件**，如`distributionUrl`属性指定的文件，其中一般包含以下内容：

1.  **Gradle 核心程序**：包括 Gradle Wrapper（`gradlew`脚本）、Gradle 执行程序（`gradle`命令）、Gradle 的运行时库以及其他必要的执行文件。
2.  **依赖库**：Gradle 使用的依赖库，用于支持构建过程中需要的各种功能和任务。
3.  **标准插件**：Gradle 标准插件，例如 Java 插件、Groovy 插件、War 插件等，用于执行常见的构建任务和操作。
4.  **其他资源文件**：可能包含一些配置文件、示例代码、文档等其他与构建相关的资源。



**这里需要理解一下什么是`GRADLE_USER_HOME`？**

具体来说，`GRADLE_USER_HOME`，默认情况下，

-   在 Unix/Linux/macOS 系统上，默认路径是 `/home/username/.gradle`。
-   在 Windows 系统上，默认路径是 `C:\Users\username\.gradle`。



区分：`zipStore* `与` distribution*`，

-   `zipStore* ` 是存放 zip 文件的位置，
-   ` distribution*` 是存放解压后文件的位置





## 命令行基础



在命令行上执行 Gradle 符合以下结构：

```shell
gradle [任务名称...] [--选项名称...]
```



### 基础命令



-   **构建项目**：
    -   `gradle build`：执行项目的构建，包括编译源代码、**运行单元测试**、打包生成可执行文件等。
    -   `gradle assemble`：编译和打包项目，生成项目的可执行文件或库文件，**但不运行单元测试**。assemble 组装的意思
-   **运行项目**：
    -   `gradle run`：运行项目的主类或指定的入口类。
    -   `gradle bootRun`：对于 Spring Boot 项目，使用 Spring Boot 插件运行项目。
-   **清理项目**：
    -   `gradle clean`：清理项目的构建输出，包括删除生成的编译文件、打包文件等。
-   **依赖管理**：
    -   `gradle dependencies`：显示项目的依赖关系。
    -   `gradle dependencyInsight --dependency <dependency>`：显示特定依赖的详细信息和依赖树。
-   **任务管理**：
    -   `gradle tasks`：列出项目中所有可执行的任务。
    -   `gradle <taskName>`：执行指定的任务，例如 `gradle compileJava` 执行编译 Java 源代码任务。
-   **测试任务**：
    -   `gradle test`：运行项目的单元测试。
    -   `gradle integrationTest`：运行项目的集成测试。
-   **发布任务**：
    -   `gradle publish`：发布项目到 Maven 仓库或其他仓库，需要配置相应的发布任务。
-   **其他常用命令**：
    -   `gradle help`：显示 Gradle 帮助信息。
    -   `gradle wrapper`：生成 Gradle Wrapper（用于在项目中使用 Gradle Wrapper）。



## 设置文件基础



![gradle basic 3](https://docs.gradle.org/8.5/userguide/img/gradle-basic-3.png)



`settings.gradle` 设置文件是每个`gradle`项目的入口点，设置文件的主要目的是向 your build 添加 subprojects，Gradle 支持单个和多项目构建。

-   对于单个项目构建，设置文件是**可选的**。
-   对于多项目构建，设置文件是强制性的，并声明所有子项目。

设置脚本支持 groovy DSL 和 kotlin DSL 



```
# 设置.gradle

# 设置项目名称
rootProject.name = 'root-project'   

# 通过添加子项目来定义项目的结构
include('sub-project-a')            
include('sub-project-b')
include('sub-project-c')
```



## 构建文件基础



![gradle basic 4](https://docs.gradle.org/8.5/userguide/img/gradle-basic-4.png)



每个 gradle 构建至少包含一个构建脚本

构建脚本中，可以添加两种类型的依赖：

-   Gradle 和 构建脚本依赖的 libraries or/and plugins
-   项目源码依赖的库



构建脚本可以是`build.gradle`用 Groovy 编写的文件，也可以是`build.gradle.kts`用 Kotlin 编写的文件。



举例介绍一下`build.gradle`文件（这是springboot的默认配置文件）

```groovy
// 插件
plugins {
    // 提供 springboot 的支持，允许使用springboot的特性，如自动配置和运行SpringBoot的应用 
    id 'org.springframework.boot' version '2.6.13'
		// 提供对Maven依赖管理的支持，允许使用Maven的DependecyManagement块来管理依赖版本
    id 'io.spring.dependency-management' version '1.0.15.RELEASE'
    // 应用java插件，添加对java编译和测试的支持
    id 'java'
}

// 定义项目的组名，通常是反向组名
group = 'com.huawei.campus'
// 定义项目的版本号
version = '0.0.1-SNAPSHOT'

// 使用 JDK 17 进行编译时，指定 sourceCompatibility 和 targetCompatibility 为 1.8 表示代码将被编译成与 Java 8 兼容的字节码。这种设置通常用于确保生成的字节码能够在 Java 8 环境中运行，尽管实际编译时使用的是更高版本的 JDK
// Compatibility 兼容性
sourceCompatibility = '1.8'

// 定义项目使用的仓库，这里使用Maven Central仓库来解析项目的依赖
repositories {
    mavenCentral()
}

// 定义项目依赖
dependencies {
  	// 引入项目在编译和运行时所需的依赖。这里引入了 spring-boot-starter，它包含了 Spring Boot 应用的基本依赖。
    implementation 'org.springframework.boot:spring-boot-starter'
  
  	// 引入项目在测试时所需的依赖。这里引入了 spring-boot-starter-test，它包含了用于 Spring Boot 应用测试的依赖。
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// 配置名为 test 的任务
tasks.named('test') {
  	// 指定使用 JUnit Platform 来运行测试，这通常用于运行 JUnit 5 的测试。
    useJUnitPlatform()
}


```



如果我们要更改默认的maven地址呢？

```groovy
// 支持多个maven仓库，gradle 会按照定义的顺序依次查找依赖项
repositories {
    mavenCentral() // 默认的 Maven Central 仓库
    maven {
        url "https://my.custom.maven.repo/maven2"
    }
    maven {
        url "https://another.custom.repo"
    }
  	maven {
        url "https://repo.mycompany.com/maven2"
        credentials {
            username 'myUsername'
            password 'myPassword'
        }
    }
}
```



我们应该从哪里查询支持的插件？

可以访问这个地址查询： https://plugins.gradle.org/



## 依赖管理基础



![gradle basic 7](https://docs.gradle.org/8.5/userguide/img/gradle-basic-7.png)

Gradle 具有对 Dependency Management 的内建支持。

依赖管理是一种声明和解析项目所需外部资源的自动化技术。

Gradle 构建脚本定义了构建可能需要外部依赖项的项目的过程。依赖项是指支持构建项目的 JAR、插件、库或源代码。

### 版本目录



**`libs.versions.toml` 文件是 Gradle 7.0 引入的一个新功能，用于管理项目中使用的库的版本信息。这个文件的作用类似于 Maven 中的 `dependencyManagement` 或 Gradle 中的 `dependencyConstraints`，可以帮助统一管理项目中各个库的版本，提高依赖管理的灵活性和可维护性。**

版本目录提供了一种将依赖项声明集中在`libs.versions.toml`文件中的方法。文件路径： `gradle/libs.versions.toml` 

目录使子项目之间的依赖关系和版本配置共享变得简单。它还允许团队在大型项目中强制执行库和插件的版本。

版本目录通常包含四个部分：

1.  `[versions]` 声明插件和库将引用的版本号。这个地方主要用于定义全局的版本信息，便于在项目的其他地方引用

    ```toml
    [versions]
    guava = "30.1.1-jre"
    junit = "5.8.2"
    logback = "1.2.6"
    ```

    在项目的`build.gradle`文件中，可以通过`${versions.guava}`的方式引用上述版本信息，如：

    ```groovy
    dependencies {
        implementation "com.google.guava:guava:${versions.guava}"
        testImplementation "org.junit.jupiter:junit-jupiter-api:${versions.junit}"
        implementation "ch.qos.logback:logback-classic:${versions.logback}"
    }
    ```

    **和maven有所不同的是，gradle不支持省略版本号。**

2.  `[libraries]` 定义构建文件中使用的库的信息，包括库的名称，版本等。在Gradle构建文件中，可以直接引用`[libraries]` 中定义的库信息

    示例：

    ```toml
    [libraries]
    guava = "com.google.guava:guava:${versions.guava}"
    junit = "org.junit.jupiter:junit-jupiter-api:${versions.junit}"
    logback = "ch.qos.logback:logback-classic:${versions.logback}"
    ```

    在项目的 `build.gradle` 文件中，可以直接使用上述库定义，如：

    ```groovy
    dependencies {
        implementation libs.guava
        testImplementation libs.junit
        implementation libs.logback
    }
    ```

3.  `[bundles]` 定义一组依赖项，通常用于管理多个相关联的依赖项。

    示例：

    ```toml
    [versions]
    springBoot = "2.6.3"
    
    [bundles]
    spring = [
        "org.springframework.boot:spring-boot-starter:${versions.springBoot}",
        "org.springframework.boot:spring-boot-starter-test:${versions.springBoot}"
    ]
    ```

    在项目的 `build.gradle` 文件中，可以直接使用 `[bundles]` 部分定义的依赖组名来引用依赖项，如：

    ```groovy
    dependencies {
        implementation bundles.spring
    }
    ```

    这样可以简化对一组相关依赖项的管理和引用。

4.  `[plugins] `定义插件。

    定义插件信息

    ```toml
    [plugins]
    java = "org.gradle.java"
    ```

    在项目的 `build.gradle` 文件中，可以直接使用上述定义的插件信息，如：

    ```groovy
    plugins {
        id plugins.java version plugins.java
    }
    ```

    

```
[versions]
androidGradlePlugin = "7.4.1"
mockito = "2.16.0"

[libraries]
google-material = { group = "com.google.android.material", name = "material", version = "1.1.0-alpha05" }
mockito-core = { module = "org.mockito:mockito-core", version.ref = "mockito" }

[plugins]
android-application = { id = "com.android.application", version.ref = "androidGradlePlugin" }
```



以下是 `libs.versions.toml` 文件的一些特点和作用：

1.  **集中管理版本信息**： `libs.versions.toml` 文件集中存放项目中使用的所有库的版本信息，使得版本管理更加清晰和统一。
2.  **约束依赖版本**： 通过 `libs.versions.toml` 文件中定义的版本信息，可以约束项目中依赖库的版本，确保各个依赖的版本与指定的版本一致。
3.  **简化依赖声明**： 在项目的 `build.gradle` 文件中，可以直接引用 `libs.versions.toml` 文件中定义的版本信息，而无需在每个依赖声明中指定版本号，从而简化了依赖声明的工作。
4.  **支持多模块项目**： **对于多模块项目，可以在根项目的 `libs.versions.toml` 文件中管理所有模块共享的库的版本信息，也可以在各个子模块的 `libs.versions.toml` 文件中管理模块私有的库的版本信息，提高了项目结构的灵活性。**
5.  **易于维护和更新**： 由于版本信息集中管理，当需要更新或切换版本时，只需修改 `libs.versions.toml` 文件中的版本号即可，无需逐个修改每个依赖的版本号，减少了维护工作的复杂度。





## 任务基础



略



## 插件基础知识



![gradle 基础 6](https://docs.gradle.org/8.5/userguide/img/gradle-basic-6.png)

### 简介



大多数功能都是通过插件添加的。使用插件也是组织构建逻辑的主要机制。插件可以提供有用的任务，例如运行代码，创建文档，设置源文件，发布档案等功能。**将插件应用**到项目可以让插件扩展项目和 Gradle 的功能。

例如：

-   Spring Boot Gradle 插件，`org.springframework.boot`提供 Spring Boot 支持。
-   Google Services Gradle 插件`com.google.gms:google-services`可在您的 Android 应用程序中启用 Google API 和 Firebase 服务。
-   Gradle Shadow 插件，`com.github.johnrengelman.shadow`是一个生成 fat/uber JAR 并支持包重定位的插件。



### 插件发布

插件以三种方式发布

1.   **核心插件**——Gradle 开发并维护一组[核心插件](https://docs.gradle.org/8.5/userguide/plugin_reference.html#plugin_reference)。
2.   **社区插件**- Gradle 社区通过[Gradle 插件门户](https://plugins.gradle.org/)共享插件。
3.   **本地插件——Gradle 允许用户使用**[API](https://docs.gradle.org/8.5/javadoc/org/gradle/api/Plugin.html)创建自定义插件。



**您可以使用插件 id**（全局唯一标识符或名称）在构建脚本中应用插件：

```
plugins {
    id «plugin id» version «plugin version» [apply «false»]
}
```





# 基础知识



## Gradle 项目

gradle使用两个主目录去执行和管理工作：

-   Gradle User Home directory : ``~/.gradle` or `C:\Users\<USERNAME>\.gradle`)`

    ```shell
    ├── caches 
    │   ├── 4.8 
    │   ├── 4.9 
    │   ├── ⋮
    │   ├── jars-3 
    │   └── modules-2 
    ├── daemon 
    │   ├── ⋮
    │   ├── 4.8
    │   └── 4.9
    ├── init.d 
    │   └── my-setup.gradle
    ├── jdks 
    │   ├── ⋮
    │   └── jdk-14.0.2+12
    ├── wrapper
    │   └── dists 
    │       ├── ⋮
    │       ├── gradle-4.8-bin
    │       ├── gradle-4.9-all
    │       └── gradle-4.9-bin
    └── gradle.properties 
    ```

-   project root directory

    ```shell
    ├── .gradle                 
    │   ├── 4.8                 
    │   ├── 4.9                 
    │   └── ⋮
    ├── build                   
    ├── gradle
    │   └── wrapper             
    ├── gradle.properties       
    ├── gradlew                 
    ├── gradlew.bat             
    ├── settings.gradle.kts     
    ├── subproject-one          
    |   └── build.gradle.kts    
    ├── subproject-two          
    |   └── build.gradle.kts    
    └── ⋮
    ```

    



## Gradle 生命周期



作为构建者，你可以定义任务与任务之间的依赖关系，gradle可以保证这些任务按照依赖关系的顺序执行。



### task graph



gradle 在执行任何 task 之前会构建 task graph

Gradle 的 Task Graph（任务图）是一个表示任务之间依赖关系的有向无环图（DAG）。它定义了任务的执行顺序，并确保每个任务在依赖的任务执行完毕之后执行。Task Graph 是 Gradle 构建系统的核心组件之一，它负责计算和管理任务的执行顺序，从而实现高效的增量构建和任务并行执行。

![任务 dag 示例](https://docs.gradle.org/8.5/userguide/img/task-dag-examples.png)

### 构建阶段



Gradle 构建有三个不同的阶段。

![作者 gradle 1](https://docs.gradle.org/8.5/userguide/img/author-gradle-1.png)



Gradle 按顺序运行这些阶段：

1.   阶段一	初始化
     -   检测 settings.gradle 文件
     -   创建一个 Settings 实例
     -   评估 settings.gradle 以确定哪些项目（和包含的构建）组成构建。
     -   Project 为每个项目创建一个实例
2.   阶段二    配置
     -   对组成构件的每个项目中的 build.gradle(.kts) 构建脚本进行评估，了解项目的结构，依赖关系，任务配置等
     -   为要求的 tasks 创建一个 task graph
3.   阶段三    执行阶段
     -   安排并执行选定的任务
     -   任务之间的依赖关系决定了他们的执行顺序
     -   任务的执行可以并行执行



![build lifecycle example](https://docs.gradle.org/8.5/userguide/img/build-lifecycle-example.png)
