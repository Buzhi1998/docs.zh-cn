---
ms.openlocfilehash: dd7d3e445772e4b5ec148576ccd1374d56e251bd
ms.sourcegitcommit: e02d17b2cf9c1258dadda4810a5e6072a0089aee
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/01/2020
ms.locfileid: "85614371"
---
### <a name="deadlock-may-result-when-using-reentrant-services"></a>使用可重入服务时可能导致死锁

#### <a name="details"></a>详细信息

死锁可能会导致一个可重入服务，该服务将服务的实例一次限制为一个执行线程。 代码中包含以下 <xref:System.ServiceModel.ServiceBehaviorAttribute> 的服务容易遇到此问题：

```csharp
[ServiceBehavior(ConcurrencyMode = ConcurrencyMode.Reentrant)]
```

#### <a name="suggestion"></a>建议

要解决此问题，可执行以下操作：

- 将服务的并发模式设置为 <xref:System.ServiceModel.ConcurrencyMode.Single?displayProperty=nameWithType> 或 &lt;System.ServiceModel.ConcurrencyMode.Multiple?displayProperty=nameWithType&gt;。 例如：

```csharp
[ServiceBehavior(ConcurrencyMode = ConcurrencyMode.Reentrant)]
```

- 安装 .NET Framework 4.6.2 的最新更新，或升级到更高版本的 .NET Framework。 这将在 <xref:System.ServiceModel.OperationContext.Current?displayProperty=nameWithType> 中禁用 <xref:System.Threading.ExecutionContext> 的流。 此行为是可配置的；它相当于将以下应用设置添加到配置文件中：

```xml
<appSettings>
  <add key="Switch.System.ServiceModel.DisableOperationContextAsyncFlow" value="true" />
</appSettings>
```

对于 Reentrant 服务，`Switch.System.ServiceModel.DisableOperationContextAsyncFlow` 的值绝不能设置为 `false`。

| “属性”    | “值”       |
|:--------|:------------|
| 范围   | 次要       |
| Version | 4.6.2       |
| 类型    | 重定目标 |

#### <a name="affected-apis"></a>受影响的 API

- <xref:System.ServiceModel.ServiceBehaviorAttribute?displayProperty=nameWithType>
- <xref:System.ServiceModel.ConcurrencyMode.Reentrant?displayProperty=nameWithType>
