# Java

## Java 风格指南

开发者应当遵循 [Google Java 风格指南](https://google.github.io/styleguide/javaguide.html)。

## 代码分析 / Lint

我们坚信一致的风格能提升代码库的可读性与可维护性。因此我们推荐使用分析器来强制执行一致性和风格规则。

我们使用 [Checkstyle](https://github.com/checkstyle/checkstyle)，并采用[与 Azure Java SDK 相同的配置](https://github.com/Azure/azure-sdk-for-java/blob/master/eng/code-quality-reports/src/main/resources/checkstyle/checkstyle.xml)。

[FindBugs](http://findbugs.sourceforge.net/) 和 [PMD](https://pmd.github.io/) 也常被使用。

## 自动格式化

Eclipse 以及其他 Java IDE 支持自动代码格式化。如果使用 Maven，一些开发者也会用 [formatter-maven-plugin](https://github.com/revelc/formatter-maven-plugin)。

## 构建校验

重要的是在 CI 中强制执行代码风格与规则，避免任何团队成员把不符合标准的代码合入 git 仓库。如果用 Azure DevOps 构建，Azure DevOps 支持 [Maven](https://learn.microsoft.com/azure/devops/pipelines/tasks/build/maven?view=azure-devops) 和 [Gradle](https://learn.microsoft.com/azure/devops/pipelines/tasks/build/gradle?view=azure-devops) 构建任务，并在每次构建中把 [PMD](https://pmd.github.io/)、[Checkstyle](https://checkstyle.sourceforge.io/) 和 [FindBugs](http://findbugs.sourceforge.net/) 代码分析工具作为其中一部分运行。

下面是启用全部三种分析工具的 Maven 构建任务示例：

```yaml
    - task: Maven@3
    displayName: 'Maven pom.xml'
    inputs:
        mavenPomFile: '$(Parameters.mavenPOMFile)'
        checkStyleRunAnalysis: true
        pmdRunAnalysis: true
        findBugsRunAnalysis: true
```

下面是启用全部三种分析工具的 Gradle 构建任务示例：

```yaml
    - task: Gradle@2
    displayName: 'gradlew build'
    inputs:
        checkStyleRunAnalysis: true
        findBugsRunAnalysis: true
        pmdRunAnalysis: true
```

## 代码评审检查清单

除了通用的代码评审检查清单，你还应当检查以下 Java 特有的条目：

* [ ] 项目是否使用 Lambda 让代码更简洁？
* [ ] 是否使用了依赖注入（DI）？配置是否正确？
* [ ] 如果代码使用 Spring Boot，是否使用 @Inject 而不是 @Autowire？
* [ ] 代码是否正确处理异常？
* [ ] 是否使用了 [Azul Zulu OpenJDK](https://learn.microsoft.com/en-us/java/azure/jdk/java-jdk-install?view=azure-java-stable)？
* [ ] 是否使用了构建自动化与包管理工具（Gradle 或 Maven）？

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/java.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}
