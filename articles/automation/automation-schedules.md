---
title: Azure 自动化中的计划
description: 自动化计划用于安排自动启动 Azure 自动化中的 Runbook。 介绍如何创建和管理计划，以便在特定的时间或按重复计划自动启动 Runbook。
services: automation
ms.service: automation
ms.component: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 09/18/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 3d8492d2a8982c9c85bfc91867f7eb6c2da04e58
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294758"
---
# <a name="scheduling-a-runbook-in-azure-automation"></a>在 Azure 自动化中计划 Runbook

要将 Azure 自动化中的 Runbook 计划为在指定的时间启动，可以将它链接到一个或多个计划。 可以在 Azure 门户中为 runbook 配置计划，使其运行一次或者每小时或每日定期运行。 还可以将它们计划为每周、每月、每周的特定几天或每月的特定几天或每月的特定一天运行。 可将一个 Runbook 链接到多个计划，一个计划可以链接多个 Runbook。

> [!NOTE]
> 目前计划不支持 Azure Automation DSC 配置。

## <a name="windows-powershell-cmdlets"></a>Windows PowerShell Cmdlet

下表中的 cmdlet 用于在 Azure 自动化中通过 Windows PowerShell 创建和管理计划。 它们作为 [Azure PowerShell 模块](/powershell/azure/overview)的一部分提供。

| Cmdlet | Description |
|:--- |:--- |
| [Get-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/get-azurermautomationschedule) |检索计划。 |
| [New-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/new-azurermautomationschedule) |创建新计划。 |
| [Remove-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/remove-azurermautomationschedule) |删除计划。 |
| [Set-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/set-azurermautomationschedule) |设置现有计划的属性。 |
| [Get-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/get-azurermautomationscheduledrunbook) |检索计划 Runbook。 |
| [Register-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/register-azurermautomationscheduledrunbook) |将 Runbook 与计划相关联。 |
| [Unregister-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/unregister-azurermautomationscheduledrunbook) |将 Runbook 与计划取消关联。 |

## <a name="creating-a-schedule"></a>创建计划

可以使用 Azure 门户或 Windows PowerShell 为 Runbook 创建新计划。

> [!NOTE]
> 当运行新的计划作业时，Azure 自动化将在自动化帐户中使用最新模块。  为了避免影响 runbook 及其自动化进程，你应该首先测试已将计划与专用于测试的自动化帐户进行链接的 runbook。  这将验证计划的 runbook 是否继续正常运行，如果没有，则可以进一步排除故障并在将更新的 runbook 版本迁移到生产之前应用所需的任何更改。
> 自动化帐户不会自动获取模块的任何新版本，除非你通过从[模块](automation-update-azure-modules.md) 中选择“更新 Azure 模块”选项来手动更新它们。

### <a name="to-create-a-new-schedule-in-the-azure-portal"></a>在 Azure 门户中创建新计划

1. 在 Azure 门户中，从你的自动化帐户中，从左侧的“共享资源”部分下选择“计划”。
1. 单击页面顶部的“添加计划”。
1. 在“新建计划”窗格中，键入新计划的“名称”和（可选）“说明”。
1. 通过选择“一次”或“定期”来选择该计划是运行一次，还是按计划重复运行。 如果选择“一次”，请指定“开始时间”，并单击“创建”。 如果选择“定期”，请指定“开始时间”，并且在“重复间隔”中选择想要 runbook 重复运行的频率（按“小时”、按“天”、按“周”或按“月”）。
    1. 如果选择“周”，则会显示一周中可供选择的日期列表。 根据需要选择天数。 计划的第一次运行将在开始时间之后选择的第一天进行。
    2. 如果选择“月”，则会看到不同的选项。 对于“每月进行次数”选项，请选择“每月天数”或“每周天数”。 如果选择“每月天数”，则会显示一个可根据需要选择天数的日历。 如果选择当月不存在的日期（例如 31 日），则计划将不会运行。 如果希望计划在最后一天运行，请在“在月份的最后一天运行”下选择“是”。 如果选择“每周天数”，则会显示“重复间隔”选项。 选择“第一”、“第二”、“第三”、“第四”或“最后”。 最后选择一天进行重复。
1. 完成后，单击“创建”。

### <a name="to-create-a-new-schedule-with-windows-powershell"></a>使用 Windows PowerShell 创建新计划

可使用 [New-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/new-azurermautomationschedule) cmdlet 创建计划。 必须指定计划的开始时间以及运行频率。

以下示例命令演示了如何使用 Azure 资源管理器 cmdlet 创建每月 15 日和 30 日运行的计划。

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
New-AzureRMAutomationSchedule –AutomationAccountName $automationAccountName –Name `
$scheduleName -StartTime "7/01/2016 15:30:00" -MonthInterval 1 `
-DaysOfMonth Fifteenth,Thirtieth -ResourceGroupName "ResourceGroup01"
```

## <a name="linking-a-schedule-to-a-runbook"></a>将计划链接到 Runbook

可将一个 Runbook 链接到多个计划，一个计划可以链接多个 Runbook。 如果 Runbook 包含参数，则可以提供这些参数的值。 必须为所有必需参数提供值，并可以为任何可选参数提供值。 此计划每次启动 Runbook 时，都将使用这些值。 可以将同一个 Runbook 附加到另一个计划，并指定不同的参数值。

### <a name="to-link-a-schedule-to-a-runbook-with-the-azure-portal"></a>使用 Azure 门户将计划链接到 Runbook

1. 在 Azure 门户中，从你的自动化帐户中，在左侧的“流程自动化”部分下选择“Runbook”。
2. 单击要计划的 Runbook 的名称。
3. 如果 Runbook 当前未链接到计划，则系统会提供“创建新计划”或“链接到现有计划”选项。
4. 如果 Runbook 有参数，可以选择选项“修改运行设置(默认值:Azure)”，此时会显示“参数”窗格，可以在其中相应地输入信息。

### <a name="to-link-a-schedule-to-a-runbook-with-windows-powershell"></a>使用 Windows PowerShell 将计划链接到 Runbook

可使用 [Register-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/register-azurermautomationscheduledrunbook) cmdlet 链接计划。 可以使用 Parameters 参数指定 Runbook 参数的值。 有关指定参数值的详细信息，请参阅[在 Azure 自动化中启动 Runbook](automation-starting-a-runbook.md)。
以下示例命令演示了如何使用带参数的 Azure 资源管理器 cmdlet 将计划链接到 Runbook。

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$runbookName = "Test-Runbook"
$scheduleName = "Sample-DailySchedule"
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Register-AzureRmAutomationScheduledRunbook –AutomationAccountName $automationAccountName `
–Name $runbookName –ScheduleName $scheduleName –Parameters $params `
-ResourceGroupName "ResourceGroup01"
```

## <a name="scheduling-runbooks-more-frequently"></a>更频繁地计划 Runbook

可为 Azure 自动化中的计划配置的最频繁间隔为一小时。 如果你需要执行比这更频繁的计划，有两个选项：

* 为 Runbook 创建 [Webhook](automation-webhooks.md)，并使用 [Azure 计划程序](../scheduler/scheduler-get-started-portal.md)调用 Webhook。 定义计划时，Azure 计划程序可提供更细的粒度。

* 创建四个计划，全部每隔一小时运行一次，每个运行 15 分钟。 此方案使用不同的计划，可让 Runbook 每隔 15 分钟运行一次。

## <a name="disabling-a-schedule"></a>禁用计划

禁用计划后，链接到该计划的所有 Runbook 不再按该计划运行。 可以手动禁用计划，也可以在创建带频率的计划时设置其过期时间。 到达过期时间时，会禁用该计划。

### <a name="to-disable-a-schedule-from-the-azure-portal"></a>从 Azure 门户禁用计划

1. 在 Azure 门户中，从你的自动化帐户中，从左侧的“共享资源”部分下选择“计划”。
1. 单击某个计划的名称以打开详细信息窗格。
1. 将“已启用”更改为“否”。

### <a name="to-disable-a-schedule-with-windows-powershell"></a>使用 Windows PowerShell 禁用计划

可使用 [Set-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/set-azurermautomationschedule) cmdlet 更改现有计划的属性。 若要禁用计划，请为 **IsEnabled** 参数指定 **false**。

以下示例命令演示了如何使用 Azure 资源管理器 cmdlet 禁用 Runbook 的计划。

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Set-AzureRmAutomationSchedule –AutomationAccountName $automationAccountName `
–Name $scheduleName –IsEnabled $false -ResourceGroupName "ResourceGroup01"
```

## <a name="next-steps"></a>后续步骤

* 若要在 Azure 自动化中开始使用 Runbook，请参阅[在 Azure 自动化中启动 Runbook](automation-starting-a-runbook.md)
