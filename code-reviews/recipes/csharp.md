# C\#

## 风格指南

开发者应当遵循微软的 [C# 编码约定](https://learn.microsoft.com/dotnet/csharp/fundamentals/coding-style/coding-conventions)，并在适用时遵循微软的[安全编码指南](https://learn.microsoft.com/dotnet/standard/security/secure-coding-guidelines)。

## 代码分析 / Lint

我们坚信一致的风格能提升代码库的可读性与可维护性。因此我们推荐使用分析器 / linter 来强制执行一致性和风格规则。

### 项目配置

我们推荐为你的解决方案建立一套通用配置，在解决方案的所有项目中引用它。创建一个 `common.props` 文件，包含所有项目的默认值：

```xml
<Project>
...
    <ItemGroup>
        <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" Version="5.0.3">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <PackageReference Include="StyleCop.Analyzers" Version="1.1.118">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
    </ItemGroup>
    <PropertyGroup>
        <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    </PropertyGroup>
    <ItemGroup Condition="Exists('$(MSBuildThisFileDirectory)../.editorconfig')" >
        <AdditionalFiles Include="$(MSBuildThisFileDirectory)../.editorconfig" />
    </ItemGroup>
...
</Project>
```

然后你可以在其他项目文件中引用 `common.props`，确保配置一致。

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <Import Project="..\common.props" />
</Project>
```

[.editorconfig](https://learn.microsoft.com/en-us/visualstudio/ide/editorconfig-code-style-settings-reference?view=vs-2019) 允许配置和覆盖规则。你可以在项目级放置 .editorconfig 文件，为不同项目（例如测试项目）定制规则。

[关于各类规则配置的详细信息](https://learn.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2019)。

### .NET 分析器

微软的 .NET 分析器基于 .NET 编译器平台（Roslyn）实现了代码质量规则和 .NET API 用法规则。它取代了微软旧的 FxCop 分析器。

[启用或安装第一方 .NET 分析器](https://learn.microsoft.com/en-us/visualstudio/code-quality/install-net-analyzers?view=vs-2019)。

如果你目前在使用旧的 FxCop 分析器，请[从 FxCop 分析器迁移到 .NET 分析器](https://learn.microsoft.com/en-us/visualstudio/code-quality/migrate-from-fxcop-analyzers-to-net-analyzers?view=vs-2019)。

### StyleCop 分析器

StyleCop 分析器是一个 NuGet 包（StyleCop.Analyzers），可以安装到任意项目中。它主要关注代码风格规则，确保团队遵循同一套规则，而不用为花括号和空格进行主观争论。详细信息见：[适用于 .NET 编译器平台的 StyleCop 分析器](https://github.com/DotNetAnalyzers/StyleCopAnalyzers)。

团队至少应当采用[托管推荐规则](https://learn.microsoft.com/en-us/visualstudio/code-quality/managed-minimum-rules-rule-set-for-managed-code?view=vs-2022)规则集。

## 自动格式化

用 .editorconfig 在项目中配置代码格式化规则。

## 构建校验

重要的是在 CI 中强制执行代码风格与规则，避免任何团队成员把不符合标准的代码合入 git 仓库。

如果你使用 FxCop 分析器和 StyleCop 分析器，在 CI 中启用它们非常简单。你需要确保项目是通过 NuGet 和 .editorconfig 配置的（见上文“项目配置”）。配置好之后，你需要在流水线中配置构建代码的步骤，基本就这些。FxCop 分析器会运行并在构建流水线中报告结果。如果有规则被违反，你的构建会变红。

```yaml
    - task: DotNetCoreCLI@2
      displayName: 'Style Check & Build'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
```

## 在 VS Code 中启用 Roslyn 支持

只要你为 OmniSharp 启用了 Roslyn 支持，上述步骤在 VS Code 中同样有效。相关设置是 `omnisharp.enableRoslynAnalyzers`，必须设为 `true`。启用该设置后，必须“Restart Omnisharp”（可以在 VS Code 的命令面板中操作，或重启 VS Code）。

![rosyln-support](https://2540885121-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FdjAcIWTODFml0GX4YWCQ%2Fuploads%2FeRXj37XIcMWc5wSoc0eb%2Fvscode-roslyn.png?alt=media)

## 代码评审检查清单

除了通用的代码评审检查清单，你还应当检查以下 C# 特有的条目：

* [ ] 代码是否正确使用了[异步编程构造](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/#BKMK_AsyncandAwait)，包括正确使用 `await`、`Task.WhenAll` 以及 CancellationToken？
* [ ] 代码是否存在并发问题？共享对象是否得到了恰当保护？
* [ ] 是否使用了依赖注入（DI）？配置是否正确？
* [ ] 项目中包含的[中间件](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/index?view=aspnetcore-2.1\&tabs=aspnetcore2x)配置是否正确？
* [ ] 资源是否通过 IDispose 模式确定性地释放？所有可释放对象是否都被正确释放（[using 模式](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-statement)）？
* [ ] 代码是否创建了大量短命对象？能否优化 GC 压力？
* [ ] 代码是否以会引发装箱（boxing）的方式编写？
* [ ] 代码是否[正确处理异常](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)？
* [ ] 是否使用包管理（NuGet）而不是把 DLL 提交进仓库？
* [ ] 代码是否恰当地使用了 LINQ？为了替换一个很短的循环而引入 LINQ、或以性能不佳的方式使用它，通常都不合适。
* [ ] 代码是否恰当地校验了参数（即 [CA1062](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1062)）？可以考虑借助 [Ensure.That](https://github.com/danielwertheim/Ensure.That) 这类扩展。
* [ ] 代码是否包含遥测（[指标、追踪](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)与[日志](https://serilog.net/)）埋点？
* [ ] 代码是否通过类来利用[选项设计模式](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.1)，为一组相关设置提供强类型访问？
* [ ] 主类中是否使用常量而不是裸字符串？或者，如果这些字符串跨文件/类使用，是否有专门的静态类存放常量？
* [ ] 魔法数字是否有解释？代码中不应出现没有任何注释说明其来由的数字。如果该数字反复出现，是否有对应的常量/枚举或等价物？
* [ ] 是否设置了恰当的异常处理？捕获异常基类（`catch (Exception)`）通常不是正确的做法。应当捕获可能发生的具体异常，例如 `IOException`。
* [ ] `#pragma` 的使用是否合理？
* [ ] 测试是否按照 **Arrange/Act/Assert** 模式正确组织，并据此写了文档？
* [ ] 如果是异步方法，方法名是否以 `Async` 后缀结尾？
* [ ] 如果方法确实是异步的，是否用 `Task.Delay` 而不是 `Thread.Sleep`？`Task.Delay` 不会阻塞当前线程，它创建一个不阻塞线程即可完成的任务，因此在多线程、多任务环境中更应优先选择它。
* [ ] 异步任务是否需要取消令牌（cancellation token），而不是用 bool 标志位？
* [ ] 是否设置了最低限度的日志？使用的日志级别是否合理？
* [ ] internal / private / public 的类和方法是否使用得当？
* [ ] 自动属性的 set 和 get 是否使用得当？在没有构造函数、需要反序列化的模型中，全部可访问是可以的。对其他类而言，通常用 private set 或 internal set 更好。
* [ ] 流和其他可释放类是否使用了 `using` 模式？如果没有，最好显式调用 `Dispose` 方法。
* [ ] 在内存中维护集合的类是否线程安全？在并发场景下使用时，请使用锁模式。

{% hint style="info" %}
**非官方社区翻译** —— 本页译自 [microsoft/code-with-engineering-playbook](https://github.com/microsoft/code-with-engineering-playbook) 的 `docs/code-reviews/recipes/csharp.md`，原文档以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 许可发布。本翻译不是 Microsoft 官方版本，且可能包含改动。
{% endhint %}
