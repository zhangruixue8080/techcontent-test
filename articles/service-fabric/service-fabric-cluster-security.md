<properties
   pageTitle="保护 Service Fabric 群集 | Azure"
   description="介绍 Service Fabric 群集的安全方案，以及用于实现这些方案的各项技术。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="07/08/2016"
   wacn.date="08/08/2016"/>

# Service Fabric 群集安全方案

Service Fabric 群集是你拥有的资源。为了防止未经授权访问资源，你必须保护资源，尤其是其中有正在运行的生产工作负荷时。本文概述 Azure 或单机上运行的群集的安全方案，以及用于实现这些方案的各种技术。群集安全方案包括：

- 节点到节点安全性
- 客户端到节点安全性
- 基于角色的访问控制 (RBAC)

## 节点到节点安全性
保护群集中 VM 或计算机之间的通信。这可确保只有已获授权加入群集的计算机可以参与托管应用程序和群集中的服务。

![节点到节点通信示意图][Node-to-Node]

在 Azure 上运行的群集或在 Windows 上运行的独立群集可对 Windows Server 计算机使用[证书安全性](https://msdn.microsoft.com/zh-cn/library/ff649801.aspx)或 [Windows 安全性](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)。
### 节点到节点的证书安全性
当你创建群集时，Service Fabric 将使用指定为节点类型配置一部分的 X.509 服务器证书。本文末尾概述了这些证书是什么，以及如何获取或创建这些证书。

证书安全性是在通过 Azure 门户预览、Azure Resource Manager 模板或独立的 JSON 模板创建群集时配置的。你可以指定一个主证书和一个可选的辅助证书（用于证书滚动更新）。指定的主要证书和辅助证书应该不同于为[客户端到节点安全性](#client-to-node-security)指定的管理客户端证书和只读客户端证书。

对于 Azure，请参阅 [Secure a Service Fabric cluster on Azure using certificates](/documentation/articles/service-fabric-secure-azure-cluster-with-certs/)（使用证书保护 Azure 上的 Service Fabric 群集）或 [Set up a cluster by using an Azure Resource Manager template](/documentation/articles/service-fabric-cluster-creation-via-arm/)（使用 Azure Resource Manager 模板设置群集），以了解如何在群集中配置证书安全性。

对于独立的 Windows Server，请参阅 [Secure a standalone cluster on Windows using X.509 certificates](/documentation/articles/service-fabric-windows-cluster-x509-security/)（使用 X.509 证书保护 Windows 上的独立群集）

### 节点到节点的 Windows 安全性
对于独立的 Windows Server，请参阅 [Secure a standalone cluster on Windows using Windows security](/documentation/articles/service-fabric-windows-cluster-windows-security/)（使用 Windows 安全性保护 Windows 上的独立群集）

## 客户端到节点安全性
对客户端进行身份验证，并保护客户端与群集中单个节点之间的通信。这种类型的安全性将验证并保护客户端通信，确保只有已获授权的用户可以访问群集与群集上部署的应用程序。客户端通过其 Windows 安全性凭据或其证书安全性凭据进行唯一标识。

![客户端到节点通信示意图][Client-to-Node]

在 Azure 上运行的群集或在 Windows 上运行的独立群集可以使用[证书安全性](https://msdn.microsoft.com/zh-cn/library/ff649801.aspx)或 [Windows 安全性](https://msdn.microsoft.com/zh-cn/library/ff649396.aspx)。

### 客户端到节点的证书安全性
 客户端到节点证书安全性是在使用 Azure 门户预览、Resource Manager 模板或独立的 JSON 模板创建群集时，通过指定管理员客户端证书和/或用户客户端证书来配置的。指定的管理员客户端证书和用户客户端证书应该不同于为[节点到节点安全性](#node-to-node-security)指定的主证书和辅助证书。

客户端如果使用管理员证书连接到群集，则拥有管理功能的完全访问权限。客户端如果使用只读的用户客户端证书连接到群集，则只拥有管理功能的只读访问权限。换而言之，这些证书用于如下所述的基于角色的访问控制 (RBAC)。

对于 Azure，请参阅 [Secure a Service Fabric cluster on Azure using certificates](/documentation/articles/service-fabric-secure-azure-cluster-with-certs/)（使用证书保护 Azure 上的 Service Fabric 群集）或 [Set up a cluster by using an Azure Resource Manager template](/documentation/articles/service-fabric-cluster-creation-via-arm/)（使用 Azure Resource Manager 模板设置群集），以了解如何在群集中配置证书安全性。

对于独立的 Windows Server，请参阅 [Secure a standalone cluster on Windows using X.509 certificates](/documentation/articles/service-fabric-windows-cluster-x509-security/)（使用 X.509 证书保护 Windows 上的独立群集）

### Azure 上客户端到节点的 Azure Active Directory (AAD) 安全性
在 Azure 上运行的群集也可以使用 Azure Active Directory (AAD) 来保护对管理终结点的访问。若要了解如何创建所需的 AAD 项目、如何在创建群集时填充这些项目，以及之后如何连接到这些群集，请参阅 [Create a Service Fabric cluster using Azure Active Directory for client authentication](/documentation/articles/service-fabric-cluster-security-client-auth-with-aad/)（创建使用 Azure Active Directory 进行客户端身份验证的 Service Fabric 群集）。

## 安全建议
对于 Azure 群集，建议针对节点到节点安全性使用 AAD 安全性来验证客户端和证书。

对于独立 Windows Server 群集，如果你有 Windows Server 2012 R2 和 Active Directory，则建议结合使用 Windows 安全性和组管理帐户 (gMA)。否则，仍可以结合使用 Windows 安全性和 Windows 帐户。

## 基于角色的访问控制 (RBAC)
访问控制可让群集管理员针对不同的用户组限制特定群集操作的访问权限，使群集更加安全。连接到群集的客户端支持两种不同的访问控制类型：管理员角色和用户角色。

管理员对管理功能（包括读取/写入功能）拥有完全访问权限。默认情况下，用户只有管理功能的读取访问权限（例如查询功能），以及解析应用程序和服务的能力。

可在创建群集时为每个角色提供不同的标识（证书、AAD，等等），以指定管理员和用户客户端角色。有关默认访问控制设置以及如何更改默认设置的详细信息，请参阅 [Role based access control for Service Fabric clients](/documentation/articles/service-fabric-cluster-security-roles/)（Service Fabric 客户端的基于角色的访问控制）。


## X.509 证书和 Service Fabric
X.509 数字证书通常用于验证客户端与服务器，以及对消息进行加密和数字签名。有关这些证书的详细信息，请参阅 [Working with certificates](http://msdn.microsoft.com/zh-cn/library/ms731899.aspx)（使用证书）。

要考虑的几个要点：

- 运行生产工作负荷的群集中使用的证书应使用正确配置的 Windows Server 证书服务来创建，或者从已批准的[证书颁发机构 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 获取。
- 切勿在生产环境中使用通过 MakeCert.exe 等工具创建的临时或测试证书。
- 可以使用自签名证书，但只应使用它来测试群集，而不应在生产环境中使用。

### 服务器 X.509 证书

服务器证书的主要任务是在客户端上对服务器（节点）进行身份验证，或者在一个服务器（节点）上对另一个服务器（节点）进行身份验证。客户端或节点对节点进行身份验证时，一项初始检查是检查“使用者”字段中的公用名值。此公用名或某个证书的使用者可选名称必须存在于允许的公用名列表中。

以下文章说明了如何生成包含使用者可选名称 (SAN) 的证书：
[How to add a subject alternative name to a secure LDAP certificate](http://support.microsoft.com/zh-cn/kb/931351)（如何将使用者可选名称添加到安全的 LDAP 证书）。

“使用者”字段可以包含多个值，每个值的前面带有代表该值类型的首字母。最常见的首字母是“CN”，表示公用名，例如“CN = www.contoso.com”。“使用者”字段也可能是空白的。如果可选的“使用者可选名称”字段已填充数据，则此字段必须包含证书的公用名，以及每个使用者可选名称的一个条目。这些内容作为“DNS 名称”值输入。

证书的“预期目的”字段值应包含适当的值，例如“服务器身份验证”或“客户端身份验证”。

### 客户端 X.509 证书

客户端证书通常不由第三方证书颁发机构颁发。当前用户位置的“个人”存储通常包含由根证书颁发机构放置的客户端证书，其预期目的是“客户端身份验证”。客户端可以在需要相互身份验证时使用此类证书。

>[AZURE.NOTE] Service Fabric 群集上的所有管理操作都需要服务器证书。客户端证书不能用于管理。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->


## 后续步骤
设置群集后，可以了解群集升级：

- [Service Fabric Cluster upgrade process and expectations（Service Fabric 群集升级过程与期望）](/documentation/articles/service-fabric-cluster-upgrade/)

了解有关应用程序安全性的详细信息：

- [Application security and RunAs（应用程序安全性和 RunAs）](/documentation/articles/service-fabric-application-runas-security/)

- [Secure service communications（安全服务通信）](/documentation/articles/service-fabric-reliable-services-secure-communication/)

<!--Image references-->
[Node-to-Node]: ./media/service-fabric-cluster-security/node-to-node.png
[Client-to-Node]: ./media/service-fabric-cluster-security/client-to-node.png

<!---HONumber=Mooncake_0801_2016-->