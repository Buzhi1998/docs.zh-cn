---
title: 将 .NET Web 应用或服务迁移到 Azure 应用服务
description: 了解如何将 .NET Web 应用或服务从本地迁移到 Azure 应用服务。
ms.topic: conceptual
ms.date: 08/11/2018
ms.openlocfilehash: 57f3b981a1d94c2193160f55f9c8242da694c169
ms.sourcegitcommit: d9470d8b2278b33108332c05224d86049cb9484b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "81607875"
---
# <a name="migrate-your-net-web-app-or-service-to-azure-app-service"></a><span data-ttu-id="972a9-103">将 .NET Web 应用或服务迁移到 Azure 应用服务</span><span class="sxs-lookup"><span data-stu-id="972a9-103">Migrate your .NET web app or service to Azure App Service</span></span>

<span data-ttu-id="972a9-104">[应用服务](https://docs.microsoft.com/azure/app-service/overview)是一个完全托管的计算平台服务，经过优化，可承载可扩展的网站和 Web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="972a9-104">[App Service](https://docs.microsoft.com/azure/app-service/overview) is a fully managed compute platform service that's optimized for hosting scalable websites and web applications.</span></span> <span data-ttu-id="972a9-105">本文提供有关如何将现有应用程序提升并移向 Azure 应用服务、需要考虑的修改以及[迁移到云](https://azure.microsoft.com/migration/web-applications/)的其他资源的信息。</span><span class="sxs-lookup"><span data-stu-id="972a9-105">This article provides information on how to lift-and-shift an existing application to Azure App Service, modifications to consider, and additional resources for [moving to the cloud](https://azure.microsoft.com/migration/web-applications/).</span></span> <span data-ttu-id="972a9-106">大多数 ASP.NET 网站（Webforms、MVC）和服务（Web API、WCF）可以直接移到 Azure 应用服务而无需进行任何更改。</span><span class="sxs-lookup"><span data-stu-id="972a9-106">Most ASP.NET websites (Webforms, MVC) and services (Web API, WCF) can move directly to Azure App Service with no changes.</span></span> <span data-ttu-id="972a9-107">有些可能需要进行少量更改，而有些可能需要进行一些重构。</span><span class="sxs-lookup"><span data-stu-id="972a9-107">Some may need minor changes while others may need some refactoring.</span></span>

<span data-ttu-id="972a9-108">已准备就绪？</span><span class="sxs-lookup"><span data-stu-id="972a9-108">Ready to get started?</span></span> <span data-ttu-id="972a9-109">[将 ASP.NET + SQL 应用程序发布到 Azure 应用服务](https://tutorials.visualstudio.com/azure-webapp-migrate/intro)。</span><span class="sxs-lookup"><span data-stu-id="972a9-109">[Publish your ASP.NET + SQL application to Azure App Service](https://tutorials.visualstudio.com/azure-webapp-migrate/intro).</span></span>

## <a name="considerations"></a><span data-ttu-id="972a9-110">注意事项</span><span class="sxs-lookup"><span data-stu-id="972a9-110">Considerations</span></span>

### <a name="on-premises-resources-including-sql-server"></a><span data-ttu-id="972a9-111">本地资源（包括 SQL Server）</span><span class="sxs-lookup"><span data-stu-id="972a9-111">On-premises resources (including SQL Server)</span></span>

<span data-ttu-id="972a9-112">验证对本地资源的访问权限，因为这些资源可能需要进行迁移或更改。</span><span class="sxs-lookup"><span data-stu-id="972a9-112">Verify access to on-premises resources as these may need to be migrated or changed.</span></span> <span data-ttu-id="972a9-113">以下是用于减轻对本地资源的访问的选项：</span><span class="sxs-lookup"><span data-stu-id="972a9-113">The following are options for mitigating access to on-premises resources:</span></span>

* <span data-ttu-id="972a9-114">使用 [Azure 虚拟网络](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet)创建将应用服务连接到本地资源的 VPN。</span><span class="sxs-lookup"><span data-stu-id="972a9-114">Create a VPN connecting App Service to on-premises resources using [Azure Virtual Networks](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span></span>
* <span data-ttu-id="972a9-115">使用 [Azure 中继](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it)，在不更改防火墙的情况下，将本地服务安全地公开给云。</span><span class="sxs-lookup"><span data-stu-id="972a9-115">Securely expose on-premises services to the cloud without firewall changes using [Azure Relay](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it).</span></span>
* <span data-ttu-id="972a9-116">将依赖项（如 [SQL 数据库](https://go.microsoft.com/fwlink/?linkid=863217)）迁移到 Azure。</span><span class="sxs-lookup"><span data-stu-id="972a9-116">Migrate dependencies such as a [SQL database](https://go.microsoft.com/fwlink/?linkid=863217) to Azure.</span></span>
* <span data-ttu-id="972a9-117">在云中使用平台即服务产品来减少依赖关系。</span><span class="sxs-lookup"><span data-stu-id="972a9-117">Use platform-as-a-service offerings in the cloud to reduce dependencies.</span></span> <span data-ttu-id="972a9-118">例如，不要连接到本地邮件服务器，而应考虑使用 [SendGrid](https://docs.microsoft.com/azure/sendgrid-dotnet-how-to-send-email)。</span><span class="sxs-lookup"><span data-stu-id="972a9-118">For example, rather than connect to an on-premises mail server, consider using [SendGrid](https://docs.microsoft.com/azure/sendgrid-dotnet-how-to-send-email).</span></span>

### <a name="port-bindings"></a><span data-ttu-id="972a9-119">端口绑定</span><span class="sxs-lookup"><span data-stu-id="972a9-119">Port Bindings</span></span>

<span data-ttu-id="972a9-120">Azure 应用服务仅支持用于 HTTP 的端口 80 和用于 HTTPS 通信的端口 443。</span><span class="sxs-lookup"><span data-stu-id="972a9-120">Azure App Service supports port 80 for HTTP and port 443 for HTTPS traffic.</span></span>

<span data-ttu-id="972a9-121">对于 WCF，支持以下绑定：</span><span class="sxs-lookup"><span data-stu-id="972a9-121">For WCF, the following bindings are supported:</span></span>

<span data-ttu-id="972a9-122">绑定</span><span class="sxs-lookup"><span data-stu-id="972a9-122">Binding</span></span> | <span data-ttu-id="972a9-123">说明</span><span class="sxs-lookup"><span data-stu-id="972a9-123">Notes</span></span>
--------|--------
<span data-ttu-id="972a9-124">BasicHttp</span><span class="sxs-lookup"><span data-stu-id="972a9-124">BasicHttp</span></span> |
<span data-ttu-id="972a9-125">WSHttp</span><span class="sxs-lookup"><span data-stu-id="972a9-125">WSHttp</span></span> |
<span data-ttu-id="972a9-126">WSDualHttpBinding</span><span class="sxs-lookup"><span data-stu-id="972a9-126">WSDualHttpBinding</span></span> | <span data-ttu-id="972a9-127">必须启用 [Web 套接字支持](https://docs.microsoft.com/azure/app-service/web-sites-configure)。</span><span class="sxs-lookup"><span data-stu-id="972a9-127">[Web socket support](https://docs.microsoft.com/azure/app-service/web-sites-configure) must be enabled.</span></span>
<span data-ttu-id="972a9-128">NetHttpBinding</span><span class="sxs-lookup"><span data-stu-id="972a9-128">NetHttpBinding</span></span> | <span data-ttu-id="972a9-129">必须为双工协定启用 [Web 套接字支持](https://docs.microsoft.com/azure/app-service/web-sites-configure)。</span><span class="sxs-lookup"><span data-stu-id="972a9-129">[Web socket support](https://docs.microsoft.com/azure/app-service/web-sites-configure) must be enabled for duplex contracts.</span></span>
<span data-ttu-id="972a9-130">NetHttpsBinding</span><span class="sxs-lookup"><span data-stu-id="972a9-130">NetHttpsBinding</span></span> | <span data-ttu-id="972a9-131">必须为双工协定启用 [Web 套接字支持](https://docs.microsoft.com/azure/app-service/web-sites-configure)。</span><span class="sxs-lookup"><span data-stu-id="972a9-131">[Web socket support](https://docs.microsoft.com/azure/app-service/web-sites-configure) must be enabled for duplex contracts.</span></span>
<span data-ttu-id="972a9-132">BasicHttpContextBinding</span><span class="sxs-lookup"><span data-stu-id="972a9-132">BasicHttpContextBinding</span></span> |
<span data-ttu-id="972a9-133">WebHttpBinding</span><span class="sxs-lookup"><span data-stu-id="972a9-133">WebHttpBinding</span></span> |
<span data-ttu-id="972a9-134">WSHttpContextBinding</span><span class="sxs-lookup"><span data-stu-id="972a9-134">WSHttpContextBinding</span></span> |

### <a name="authentication"></a><span data-ttu-id="972a9-135">身份验证</span><span class="sxs-lookup"><span data-stu-id="972a9-135">Authentication</span></span>

<span data-ttu-id="972a9-136">Azure 应用服务默认支持匿名身份验证，并在需要时进行表单验证。</span><span class="sxs-lookup"><span data-stu-id="972a9-136">Azure App Service supports anonymous authentication by default and Forms authentication when intended.</span></span> <span data-ttu-id="972a9-137">Windows 身份验证仅可通过集成 Azure Active Directory 和 ADFS 加以使用。</span><span class="sxs-lookup"><span data-stu-id="972a9-137">Windows authentication can be used by integrating with Azure Active Directory and ADFS only.</span></span> <span data-ttu-id="972a9-138">[详细了解如何将本地目录与 Azure Active Directory 集成](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect)。</span><span class="sxs-lookup"><span data-stu-id="972a9-138">[Learn more about how to integrate your on-premises directories with Azure Active Directory](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect).</span></span>

### <a name="assemblies-in-the-gac-global-assembly-cache"></a><span data-ttu-id="972a9-139">GAC（全局程序集缓存）中的程序集</span><span class="sxs-lookup"><span data-stu-id="972a9-139">Assemblies in the GAC (Global Assembly Cache)</span></span>

<span data-ttu-id="972a9-140">不支持此操作。</span><span class="sxs-lookup"><span data-stu-id="972a9-140">This isn't supported.</span></span> <span data-ttu-id="972a9-141">请考虑将所需的程序集复制到应用的*\bin*文件夹。</span><span class="sxs-lookup"><span data-stu-id="972a9-141">Consider copying required assemblies to the app's *\bin* folder.</span></span> <span data-ttu-id="972a9-142">无法使用安装在服务器上的自定义 *.msi*文件（例如 PDF 生成器）。</span><span class="sxs-lookup"><span data-stu-id="972a9-142">Custom *.msi* files installed on the server (for example, PDF generators) cannot be used.</span></span>

### <a name="iis-settings"></a><span data-ttu-id="972a9-143">IIS 设置</span><span class="sxs-lookup"><span data-stu-id="972a9-143">IIS settings</span></span>
<span data-ttu-id="972a9-144">以往要在应用程序中通过 applicationHost.config 配置的所有设置现在可以通过 Azure 门户进行配置。</span><span class="sxs-lookup"><span data-stu-id="972a9-144">Everything traditionally configured via applicationHost.config in your application can now be configured through the Azure portal.</span></span> <span data-ttu-id="972a9-145">这适用于 AppPool 位、启用/禁用 WebSocket、托管管道版本、.NET 框架版本 （2.0/4.0）等。</span><span class="sxs-lookup"><span data-stu-id="972a9-145">This applies to AppPool bitness, enable/disable WebSockets, managed pipeline version, .NET Framework version (2.0/4.0), and so on.</span></span> <span data-ttu-id="972a9-146">若要修改[应用程序设置](https://docs.microsoft.com/azure/app-service/web-sites-configure)，请导航到 [Azure 门户](https://portal.azure.com)，打开 Web 应用的边栏选项卡，并选择“应用程序设置”选项卡。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="972a9-146">To modify your [application settings](https://docs.microsoft.com/azure/app-service/web-sites-configure), navigate to the [Azure portal](https://portal.azure.com), open the blade for your web app, and then select the **Application Settings** tab.</span></span>

#### <a name="iis5-compatibility-mode"></a><span data-ttu-id="972a9-147">IIS5 兼容模式</span><span class="sxs-lookup"><span data-stu-id="972a9-147">IIS5 Compatibility Mode</span></span>
<span data-ttu-id="972a9-148">不支持 IIS5 兼容模式。</span><span class="sxs-lookup"><span data-stu-id="972a9-148">IIS5 Compatibility Mode is not supported.</span></span> <span data-ttu-id="972a9-149">在 Azure 应用服务中，每个 Web 应用及其下的所有应用程序都在同一工作进程中使用一组特定的[应用程序池](https://technet.microsoft.com/library/cc735247(v=WS.10).aspx)运行。</span><span class="sxs-lookup"><span data-stu-id="972a9-149">In Azure App Service, each web app and all of the applications under it run in the same worker process with a specific set of [application pools](https://technet.microsoft.com/library/cc735247(v=WS.10).aspx).</span></span>

#### <a name="iis7-schema-compliance"></a><span data-ttu-id="972a9-150">IIS7+ 架构符合性</span><span class="sxs-lookup"><span data-stu-id="972a9-150">IIS7+ schema compliance</span></span>  
<span data-ttu-id="972a9-151">Azure 应用服务 IIS 架构中未定义一些元素和属性。</span><span class="sxs-lookup"><span data-stu-id="972a9-151">Some elements and attributes are not defined in the Azure App Service IIS schema.</span></span> <span data-ttu-id="972a9-152">如果遇到问题，请考虑使用 [XDT 转换](https://azure.microsoft.com/documentation/articles/web-sites-transform-extend/)。</span><span class="sxs-lookup"><span data-stu-id="972a9-152">If you encounter issues, consider using [XDT transforms](https://azure.microsoft.com/documentation/articles/web-sites-transform-extend/).</span></span>

#### <a name="single-application-pool-per-site"></a><span data-ttu-id="972a9-153">每个站点的单个应用程序池</span><span class="sxs-lookup"><span data-stu-id="972a9-153">Single application pool per site</span></span>  
<span data-ttu-id="972a9-154">在 Azure 应用服务中，每个 Web 应用及其下的所有应用程序都在同一应用程序池中运行。</span><span class="sxs-lookup"><span data-stu-id="972a9-154">In Azure App Service, each web app and all of the applications under it run in the same application pool.</span></span> <span data-ttu-id="972a9-155">请考虑使用通用设置建立单个应用程序池，或为每个应用程序创建单独的 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="972a9-155">Consider establishing a single application pool with common settings or creating a separate web app for each application.</span></span>

### <a name="com-and-com-components"></a><span data-ttu-id="972a9-156">COM 和 COM+ 组件</span><span class="sxs-lookup"><span data-stu-id="972a9-156">COM and COM+ components</span></span>  
<span data-ttu-id="972a9-157">Azure 应用服务不允许在平台上注册 COM 组件。</span><span class="sxs-lookup"><span data-stu-id="972a9-157">Azure App Service does not allow the registration of COM components on the platform.</span></span> <span data-ttu-id="972a9-158">如果应用使用任何 COM 组件，则需要在托管代码中重写这些组件，并与站点或应用程序一起部署这些组件。</span><span class="sxs-lookup"><span data-stu-id="972a9-158">If your app makes use of any COM components, these need to be rewritten in managed code and deployed with the site or application.</span></span>

### <a name="physical-directories"></a><span data-ttu-id="972a9-159">物理目录</span><span class="sxs-lookup"><span data-stu-id="972a9-159">Physical directories</span></span>
<span data-ttu-id="972a9-160">Azure 应用服务不允许访问物理驱动器。</span><span class="sxs-lookup"><span data-stu-id="972a9-160">Azure App Service does not allow physical drive access.</span></span> <span data-ttu-id="972a9-161">您可能需要使用 Azure[文件](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)通过 SMB 访问文件。</span><span class="sxs-lookup"><span data-stu-id="972a9-161">You may need to use [Azure Files](https://docs.microsoft.com/azure/storage/files/storage-files-introduction) to access files via SMB.</span></span> <span data-ttu-id="972a9-162">[Azure Blob存储](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)可以存储文件以通过 HTTPS 进行访问。</span><span class="sxs-lookup"><span data-stu-id="972a9-162">[Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction) can store files for access via HTTPS.</span></span>

### <a name="isapi-filters"></a><span data-ttu-id="972a9-163">ISAPI 筛选器</span><span class="sxs-lookup"><span data-stu-id="972a9-163">ISAPI filters</span></span>  
<span data-ttu-id="972a9-164">Azure 应用服务可以支持使用 ISAPI 筛选器，但是，ISAPI DLL 必须与站点一起部署并通过 web.config 注册。</span><span class="sxs-lookup"><span data-stu-id="972a9-164">Azure App Service can support the use of ISAPI Filters, however, the ISAPI DLL must be deployed with your site and registered via web.config.</span></span>

### <a name="https-bindings-and-ssl"></a><span data-ttu-id="972a9-165">HTTPS 绑定和 SSL</span><span class="sxs-lookup"><span data-stu-id="972a9-165">HTTPS bindings and SSL</span></span>
<span data-ttu-id="972a9-166">不会迁移 HTTPS 绑定，也不会与您的网站关联 SSL 证书。</span><span class="sxs-lookup"><span data-stu-id="972a9-166">HTTPS bindings are not migrated, nor are the SSL certificates associated with your web sites.</span></span> <span data-ttu-id="972a9-167">但是，在完成站点迁移后，[可以手动上传 SSL证书](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)。</span><span class="sxs-lookup"><span data-stu-id="972a9-167">[SSL certificates can be manually uploaded](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl) after site migration is completed, however.</span></span>

### <a name="sharepoint-and-frontpage"></a><span data-ttu-id="972a9-168">SharePoint 和 FrontPage</span><span class="sxs-lookup"><span data-stu-id="972a9-168">SharePoint and FrontPage</span></span>
<span data-ttu-id="972a9-169">不支持 SharePoint 和 FrontPage 服务器扩展 (FPSE)。</span><span class="sxs-lookup"><span data-stu-id="972a9-169">SharePoint and FrontPage Server Extensions (FPSE) are not supported.</span></span>

### <a name="web-site-size"></a><span data-ttu-id="972a9-170">网站大小</span><span class="sxs-lookup"><span data-stu-id="972a9-170">Web site size</span></span>  
<span data-ttu-id="972a9-171">免费站点的内容大小限制为 1 GB。</span><span class="sxs-lookup"><span data-stu-id="972a9-171">Free sites have a size limit of 1 GB of content.</span></span> <span data-ttu-id="972a9-172">如果你的站点大于 1 GB，则必须升级到付费 SKU。</span><span class="sxs-lookup"><span data-stu-id="972a9-172">If your site is greater than 1 GB, you must upgrade to a paid SKU.</span></span> <span data-ttu-id="972a9-173">请参阅[应用服务定价](https://azure.microsoft.com/pricing/details/app-service/windows/)。</span><span class="sxs-lookup"><span data-stu-id="972a9-173">See [App Service pricing](https://azure.microsoft.com/pricing/details/app-service/windows/).</span></span>

### <a name="database-size"></a><span data-ttu-id="972a9-174">数据库大小</span><span class="sxs-lookup"><span data-stu-id="972a9-174">Database size</span></span>  
<span data-ttu-id="972a9-175">对于 SQL Server 数据库，请检查当前 [SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database)。</span><span class="sxs-lookup"><span data-stu-id="972a9-175">For SQL Server databases, please check the current [SQL Database pricing](https://azure.microsoft.com/pricing/details/sql-database).</span></span>

### <a name="azure-active-directory-aad-integration"></a><span data-ttu-id="972a9-176">Azure Active Directory (AAD) 集成</span><span class="sxs-lookup"><span data-stu-id="972a9-176">Azure Active Directory (AAD) integration</span></span>  
<span data-ttu-id="972a9-177">AAD 不适用于免费应用。</span><span class="sxs-lookup"><span data-stu-id="972a9-177">AAD does not work with free apps.</span></span> <span data-ttu-id="972a9-178">若要使用 AAD，必须升级应用 SKU。</span><span class="sxs-lookup"><span data-stu-id="972a9-178">To use AAD, you must upgrade the app SKU.</span></span> <span data-ttu-id="972a9-179">请参阅[应用服务定价](https://azure.microsoft.com/pricing/details/app-service/windows/)。</span><span class="sxs-lookup"><span data-stu-id="972a9-179">See [App Service pricing](https://azure.microsoft.com/pricing/details/app-service/windows/).</span></span>

### <a name="monitoring-and-diagnostics"></a><span data-ttu-id="972a9-180">监视和诊断</span><span class="sxs-lookup"><span data-stu-id="972a9-180">Monitoring and diagnostics</span></span>
<span data-ttu-id="972a9-181">当前用于监视和诊断的本地解决方案不太可能在云中工作。</span><span class="sxs-lookup"><span data-stu-id="972a9-181">Your current on-premises solutions for monitoring and diagnostics are unlikely to work in the cloud.</span></span> <span data-ttu-id="972a9-182">但是，Azure 提供了日志记录、监视和诊断工具，用于识别和调试 Web 应用的问题。</span><span class="sxs-lookup"><span data-stu-id="972a9-182">However, Azure provides tools for logging, monitoring, and diagnostics so that you can identify and debug issues with web apps.</span></span> <span data-ttu-id="972a9-183">可以轻松地在 Web 应用的配置中为其启用诊断，并可以在 Azure Application Insights 中查看记录的日志。</span><span class="sxs-lookup"><span data-stu-id="972a9-183">You can easily enable diagnostics for your web app in its configuration, and you can view the logs recorded in Azure Application Insights.</span></span> <span data-ttu-id="972a9-184">[详细了解如何为 Web 应用启用诊断日志记录](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log)。</span><span class="sxs-lookup"><span data-stu-id="972a9-184">[Learn more about enabling diagnostics logging for web apps](https://docs.microsoft.com/azure/app-service/web-sites-enable-diagnostic-log).</span></span>

### <a name="connection-strings-and-application-settings"></a><span data-ttu-id="972a9-185">连接字符串和应用程序设置</span><span class="sxs-lookup"><span data-stu-id="972a9-185">Connection strings and application settings</span></span>
<span data-ttu-id="972a9-186">请考虑使用 [Azure KeyVault](https://docs.microsoft.com/azure/key-vault/)，这是一个可安全存储应用程序中使用的敏感信息的服务。</span><span class="sxs-lookup"><span data-stu-id="972a9-186">Consider using [Azure KeyVault](https://docs.microsoft.com/azure/key-vault/), a service that securely stores sensitive information used in your application.</span></span> <span data-ttu-id="972a9-187">或者，可将此数据存储为应用服务设置。</span><span class="sxs-lookup"><span data-stu-id="972a9-187">Alternatively, you can store this data as an App Service setting.</span></span>

### <a name="dns"></a><span data-ttu-id="972a9-188">DNS</span><span class="sxs-lookup"><span data-stu-id="972a9-188">DNS</span></span>
<span data-ttu-id="972a9-189">可能需要根据应用程序的要求更新 DNS 配置。</span><span class="sxs-lookup"><span data-stu-id="972a9-189">You may need to update DNS configurations based on the requirements of your application.</span></span> <span data-ttu-id="972a9-190">可在应用服务的[自定义域设置](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)中配置这些 DNS 设置。</span><span class="sxs-lookup"><span data-stu-id="972a9-190">These DNS settings can be configured in the App Service [custom domain settings](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain).</span></span>

## <a name="azure-app-service-with-windows-containers"></a><span data-ttu-id="972a9-191">使用 Windows 容器的 Azure 应用服务</span><span class="sxs-lookup"><span data-stu-id="972a9-191">Azure App Service with Windows Containers</span></span>
<span data-ttu-id="972a9-192">如果你的应用无法直接迁移到应用服务，请考虑使用 Windows 容器的应用服务，这样可以使用 GAC、COM 组件、MSI，完全访问.NET FX API、DirectX 等。</span><span class="sxs-lookup"><span data-stu-id="972a9-192">If your app cannot be migrated directly to App Service, consider App Service using Windows Containers, which enables usage of the GAC, COM components, MSIs, full access to .NET FX APIs, DirectX, and more.</span></span>

## <a name="see-also"></a><span data-ttu-id="972a9-193">请参阅</span><span class="sxs-lookup"><span data-stu-id="972a9-193">See also</span></span>

* [<span data-ttu-id="972a9-194">如何确定应用是否符合应用服务的条件</span><span class="sxs-lookup"><span data-stu-id="972a9-194">How to determine if your app qualifies for App Service</span></span>](https://appmigration.microsoft.com/)
* [<span data-ttu-id="972a9-195">将数据库转移到云中</span><span class="sxs-lookup"><span data-stu-id="972a9-195">Moving your database to the cloud</span></span>](https://go.microsoft.com/fwlink/?linkid=863217)
* [<span data-ttu-id="972a9-196">Azure Web 应用沙盒详细信息和限制</span><span class="sxs-lookup"><span data-stu-id="972a9-196">Azure web app sandbox details and restrictions</span></span>](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)