---
title: 使用 Windows 虚拟机的资源管理器模板将来宾 OS 指标发送到 Azure Monitor 指标存储
description: 使用 Windows 虚拟机的资源管理器模板将来宾 OS 指标发送到 Azure Monitor 指标存储
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: howto
ms.date: 09/24/2018
ms.author: ancav
ms.component: metrics
ms.openlocfilehash: 4ed911766a14dd35ea662326a5d50df11cf81698
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984054"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-using-a-resource-manager-template-for-a-windows-virtual-machine"></a>使用 Windows 虚拟机的资源管理器模板将来宾 OS 指标发送到 Azure Monitor 指标存储

Azure Monitor [Windows Azure诊断扩展](azure-diagnostics.md) (WAD) 支持从作为虚拟机、云服务或 Service Fabric 群集的一部分运行的来宾操作系统（来宾 OS）中收集指标和日志。  该扩展可以将遥测发送到之前链接的文章中列出的许多不同位置。  

本文介绍将 Windows 虚拟机的来宾 OS 性能指标发送到 Azure Monitor 数据存储的过程。 自 WAD 1.11 版起，可将指标直接写入已收集标准平台指标的 Azure Monitor 指标存储。 通过将它们存储在此位置，可以执行可对平台指标执行的相同操作。  操作包括近乎实时的警报、绘制图表、路由、接受 REST API 访问等。  以前，WAD 扩展写入 Azure 存储，而不是 Azure Monitor 数据存储。   

如果不熟悉资源管理器模板，请了解[模板部署](../azure-resource-manager/resource-group-overview.md)及其结构和语法。  

## <a name="pre-requisites"></a>先决条件

- 必须向 [Microsoft.Insights](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1) 注册订阅 

- 需要安装 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview?view=azurermps-6.8.1)，也可以使用 [Azure CloudShell](https://docs.microsoft.com/azure/cloud-shell/overview.md) 

 
## <a name="set-up-azure-monitor-as-a-data-sink"></a>将 Azure Monitor 设置为数据接收器 
Azure 诊断扩展使用名为“数据接收器”的功能将指标和日志路由到不同位置。  以下步骤演示如何通过资源管理器模板和 PowerShell 来使用新的“Azure Monitor”数据接收器部署 VM。 

## <a name="author-resource-manager-template"></a>创建资源管理器模板 
对于此示例，可使用公开发布的示例模板。 起始模板位于 https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows 

- Azuredeploy.json 是用于部署虚拟机的预配置资源管理器模板。 

- Azuredeploy.parameters.json 是参数文件，用于存储要为 VM 设置的用户名和密码等信息。 部署期间，资源管理器模板使用此文件中设置的参数。 

下载这两个文件并在本地保存。 

###  <a name="modify-azuredeployparametersjson"></a>修改 azuredeploy.parameters.json
打开 azuredeploy.parameters.json 文件 

1. 输入 VM 的“adminUsername”和“adminPassword”的值。 这些参数用于对 VM 进行远程访问。 请勿使用此模板中的参数以避免 VM 被劫持。 机器人在 Internet 上扫描公共 Github 存储库中的用户名和密码。 它们可能会使用这些默认值测试 VM。  

1. 为 VM 创建唯一 dnsname。  

### <a name="modify-azuredeployjson"></a>修改 azuredeploy.json

打开 azuredeploy.json 文件 

输入“storageAccountName”后，将存储帐户 ID 添加到模板的“变量”部分。  

```json
// Find these lines 
"variables": { 
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'sawinvm')]", 

// Add this line directly below.  
    "accountid": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]", 
```

将此托管服务标识 (MSI) 扩展添加到“资源”部分顶部的模板。  该扩展可确保 Azure Monitor 接受所发出的指标。  

```json
//Find this code 
"resources": [
// Add this code directly below
    { 
        "type": "Microsoft.Compute/virtualMachines/extensions", 
        "name": "WADExtensionSetup", 
        "apiVersion": "2015-05-01-preview", 
        "location": "[resourceGroup().location]", 
        "dependsOn": [ 
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]" ], 
        "properties": { 
            "publisher": "Microsoft.ManagedIdentity", 
            "type": "ManagedIdentityExtensionForWindows", 
            "typeHandlerVersion": "1.0", 
            "autoUpgradeMinorVersion": true, 
            "settings": { 
                "port": 50342 
            } 
        } 
    }, 
```

将“标识”配置添加到 VM 资源，以确保 Azure 为 MSI 扩展分配系统标识。 此步骤可确保 VM 可以将有关自身的来宾指标发送到 Azure Monitor 

```json
// Find this section
                "subnet": {
            "id": "[variables('subnetRef')]"
            }
        }
        }
    ]
    }
},
{ 
    "apiVersion": "2017-03-30", 
    "type": "Microsoft.Compute/virtualMachines", 
    "name": "[variables('vmName')]", 
    "location": "[resourceGroup().location]", 
    // add these 3 lines below
    "identity": {  
    "type": "SystemAssigned" 
    }, 
    //end of added lines
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
    "hardwareProfile": {   
    ...
```

添加以下配置以在 Windows 虚拟机上启用诊断扩展。  对于基于资源管理器的简单虚拟机，可将扩展配置添加到该虚拟机的 resources 数组。 行“接收器”：“AzMonSink”以及该部分后面的相应“SinksConfig”使扩展可以直接向 Azure Monitor 发出指标。 根据需要任意添加/删除性能计数器。  


```json
        "networkProfile": {
            "networkInterfaces": [
            {
                "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
            }
            ]
        },
"diagnosticsProfile": { 
    "bootDiagnostics": { 
    "enabled": true, 
    "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]" 
    } 
} 
}, 
//Start of section to add 
"resources": [        
{ 
            "type": "extensions", 
            "name": "Microsoft.Insights.VMDiagnosticsSettings", 
            "apiVersion": "2015-05-01-preview", 
            "location": "[resourceGroup().location]", 
            "dependsOn": [ 
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]" 
            ], 
            "properties": { 
            "publisher": "Microsoft.Azure.Diagnostics", 
            "type": "IaaSDiagnostics", 
            "typeHandlerVersion": "1.4", 
            "autoUpgradeMinorVersion": true, 
            "settings": { 
                "WadCfg": { 
                "DiagnosticMonitorConfiguration": { 
    "overallQuotaInMB": 4096, 
    "DiagnosticInfrastructureLogs": { 
                    "scheduledTransferLogLevelFilter": "Error" 
        }, 
                    "Directories": { 
                    "scheduledTransferPeriod": "PT1M", 
    "IISLogs": { 
                        "containerName": "wad-iis-logfiles" 
                    }, 
                    "FailedRequestLogs": { 
                        "containerName": "wad-failedrequestlogs" 
                    } 
                    }, 
                    "PerformanceCounters": { 
                    "scheduledTransferPeriod": "PT1M", 
                    "sinks": "AzMonSink", 
                    "PerformanceCounterConfiguration": [ 
                        { 
                        "counterSpecifier": "\\Memory\\Available Bytes", 
                        "sampleRate": "PT15S" 
                        }, 
                        { 
                        "counterSpecifier": "\\Memory\\% Committed Bytes In Use", 
                        "sampleRate": "PT15S" 
                        }, 
                        { 
                        "counterSpecifier": "\\Memory\\Committed Bytes", 
                        "sampleRate": "PT15S" 
                        } 
                    ] 
                    }, 
                    "WindowsEventLog": { 
                    "scheduledTransferPeriod": "PT1M", 
                    "DataSource": [ 
                        { 
                        "name": "Application!*" 
                        } 
                    ] 
                    }, 
                    "Logs": { 
                    "scheduledTransferPeriod": "PT1M", 
                    "scheduledTransferLogLevelFilter": "Error" 
                    } 
                }, 
                "SinksConfig": { 
                    "Sink": [ 
                    { 
                        "name" : "AzMonSink", 
                        "AzureMonitor" : {} 
                    } 
                    ] 
                } 
                }, 
                "StorageAccount": "[variables('storageAccountName')]" 
            }, 
            "protectedSettings": { 
                "storageAccountName": "[variables('storageAccountName')]", 
                "storageAccountKey": "[listKeys(variables('accountid'),'2015-06-15').key1]", 
                "storageAccountEndPoint": "https://core.windows.net/" 
            } 
            } 
        } 
        ] 
//End of section to add 
```


保存和关闭这两个文件 
 

## <a name="deploy-the-resource-manager-template"></a>部署资源管理器模板 

> [!NOTE]
> 必须运行 Azure 诊断扩展 1.5 版或更高版本，并在资源管理器模板中将 "autoUpgradeMinorVersion": 属性设置为“true”。  Azure 会在启动 VM 后加载适当的扩展。 如果模板中没有这些设置，请进行更改并重新部署模板。 


我们将利用 Azure PowerShell 部署资源管理器模板。  

1. 启动 PowerShell 
1. 使用 `Login-AzureRmAccount` 登录到 Azure
1. 使用 `Get-AzureRmSubscription` 获取订阅列表
1. 设置将在其中创建/更新虚拟机的订阅 

   ```PowerShell
   Select-AzureRmSubscription -SubscriptionName "<Name of the subscription>" 
   ```
1. 为要部署的 VM 创建新资源组，运行以下命令 

   ```PowerShell
    New-AzureRmResourceGroup -Name "<Name of Resource Group>" -Location "<Azure Region>" 
   ```
   > [!NOTE] 
   > 请记住[使用为自定义指标启用的 Azure 区域](metrics-custom-overview.md)。 
 
1. 执行以下命令以使用以下项部署 VM  
   > [!NOTE] 
   > 如果要更新现有 VM，只需将“-Mode Incremental”添加到以下命令的末尾。 
 
   ```PowerShell
   New-AzureRmResourceGroupDeployment -Name "<NameThisDeployment>" -ResourceGroupName "<Name of the Resource Group>" -TemplateFile "<File path of your Resource Manager template>" -TemplateParameterFile "<File path of your parameters file>" 
   ```
  
1. 部署成功后，应能在 Azure 门户中找到 VM，并且 VM 应该向 Azure Monitor 发出指标。 

   > [!NOTE] 
   > 可能会遇到与所选 vmSkuSize 相关的错误。 如果发生这种情况，请返回到 azuredeploy.json 文件并更新 vmSkuSize 参数的默认值。 在这种情况下，建议尝试使用 “Standard_DS1_v2”。 

## <a name="chart-your-metrics"></a>绘制指标图表 

1. 登录 Azure 门户 

1. 在左侧菜单，单击“监视” 

1. 在“监视”页面，单击“指标”。 

   ![“指标”页](./media/metrics-store-custom-rest-api/metrics.png) 

1. 将聚合时限更改为“最近 30 分钟”。  

1. 在资源下拉列表中选择刚刚创建的 VM。 如果未更改模板中的名称，则名称应为“SimpleWinVM2”。  

1. 在命名空间下拉列表中选择“azure.vm.windows.guest” 

1. 在指标下拉列表中，选择“内存”\%“已提交的使用字节数”。  
 

## <a name="next-steps"></a>后续步骤
- 详细了解[自定义指标](metrics-custom-overview.md)。