---
title: 安装 Durable Functions 扩展和示例 - Azure
description: 了解如何针对门户开发或 Visual Studio 开发安装 Azure Functions 的 Durable Functions 扩展。
services: functions
author: kashimiz
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 10/23/2018
ms.author: azfuncdf
ms.openlocfilehash: 6bbf232fc17b9acfd4e8cd84a0cb1346ab8ea9b5
ms.sourcegitcommit: c2c279cb2cbc0bc268b38fbd900f1bac2fd0e88f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49986800"
---
# <a name="install-the-durable-functions-extension-and-samples-azure-functions"></a>安装 Durable Functions 扩展和示例 (Azure Functions)

Azure Functions 的 [Durable Functions](durable-functions-overview.md) 扩展是在 NuGet 包 [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask) 中提供的。 本文展示了如何为以下开发环境安装包和一组示例：

* Visual Studio 2017（推荐用于 C#） 
* Visual Studio Code（推荐用于 JavaScript）
* Azure 门户

## <a name="visual-studio-2017"></a>Visual Studio 2017

在开发使用 Durable Functions 的应用方面，Visual Studio 当前提供了最佳体验。  函数可以在本地运行，也可以发布到 Azure。 可以从空项目开始，也可以从一组示例函数开始。

### <a name="prerequisites"></a>先决条件

* 安装[最新版 Visual Studio](https://www.visualstudio.com/downloads/)（版本 15.6 或更高版本）。 在安装选项中包括 **Azure 开发**工作负荷。

### <a name="start-with-sample-functions"></a>从示例函数开始 

1. 下载 [Visual Studio 的示例应用 .zip 文件](https://azure.github.io/azure-functions-durable-extension/files/VSDFSampleApp.zip)。 不需要添加 NuGet 引用，因为示例项目中已包含它。
2. 安装并运行 [Azure 存储模拟器](https://docs.microsoft.com/azure/storage/storage-use-emulator)版本 5.6 或更高版本。 或者，也可以使用实际 Azure 存储连接字符串更新 local.settings.json 文件。
3. 在 Visual Studio 2017 中打开项目。 
4. 若要获得有关如何运行示例的说明，请首先查看[函数链接 - Hello 序列示例](durable-functions-sequence.md)。 示例可以在本地运行，也可以发布到 Azure。

### <a name="start-with-an-empty-project"></a>从空项目开始
 
按照与从示例开始相同的说明执行操作，但请执行以下步骤而非下载 *.zip* 文件：

1. 创建一个 Function App 项目。
2. 使用“管理 NuGet 包”搜索以下 NuGet 包引用，并将它添加到项目中：Microsoft.Azure.WebJobs.Extensions.DurableTask v1.6.2
   
## <a name="visual-studio-code"></a>Visual Studio Code

Visual Studio Code 提供一种涵盖所有主要平台（Windows、macOS 和 Linux）的本地开发体验。  函数可以在本地运行，也可以发布到 Azure。 可以从空项目开始，也可以从一组示例函数开始。

### <a name="prerequisites"></a>先决条件

* 安装[最新版本的 Visual Studio Code](https://code.visualstudio.com/Download) 

* 遵照[在本地对 Azure Functions 进行编程和测试](https://docs.microsoft.com/azure/azure-functions/functions-run-local)中“安装 Azure Functions Core Tools”下面的说明操作

    >[!IMPORTANT]
    > 如果已安装 Azure Functions 跨平台工具，请将其更新到最新可用版本。

    >[!IMPORTANT]
    >JavaScript 中的 Durable Functions 需要 Azure Functions Core Tools 2.x 版。

*  如果使用的是 Windows 计算机，请安装并运行 [Azure 存储模拟器](https://docs.microsoft.com/azure/storage/storage-use-emulator)版本 5.6 或更高版本。 或者，也可以使用实际 Azure 存储连接字符串更新 local.settings.json 文件。 


### <a name="start-with-sample-functions"></a>从示例函数开始

#### <a name="c"></a>C#

1. 克隆 [Durable Functions 存储库](https://github.com/Azure/azure-functions-durable-extension.git)。
2. 在计算机上导航到 [C# 脚本示例文件夹](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx)。 
3. 在命令提示/终端窗口中运行以下命令，安装 Azure Functions Durable Extension：

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.DurableTask -v 1.6.2
    ```
4. 在命令提示/终端窗口中运行以下命令，安装 Azure Functions Twilio Extension：

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.Twilio -v 3.0.0
    ```
5. 运行 Azure 存储模拟器，或使用实际 Azure 存储连接字符串更新 local.settings.json 文件。
6. 在 Visual Studio Code 中打开项目。 
7. 若要获得有关如何运行示例的说明，请首先查看[函数链接 - Hello 序列示例](durable-functions-sequence.md)。 示例可以在本地运行，也可以发布到 Azure。
8. 在命令提示/终端中运行以下命令以启动项目：
    ```bash
    func host start
    ```

#### <a name="javascript-functions-v2-only"></a>JavaScript（仅限 Functions v2）

1. 克隆 [Durable Functions 存储库](https://github.com/Azure/azure-functions-durable-extension.git)。
2. 在计算机上导航到 [JavaScript 示例文件夹](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/javascript)。 
3. 在命令提示符/终端窗口中运行下面的命令，以安装 Azure Functions 持久扩展

    ```bash
    func extensions install
    ```
    > [!NOTE] 
    > 这要求必须在计算机上安装 [.NET Core SDK](https://www.microsoft.com/net/download)
4. 在命令提示/终端窗口中运行以下命令，还原 npm 包：
    
    ```bash
    npm install
    ``` 
5. 使用 Azure 存储帐户中的 `AzureWebJobsStorage` 连接字符串更新 local.settings.json 文件。  此存储帐户用于持久函数状态。
6. 在 Visual Studio Code 等编辑器中打开项目。 
7. 若要获得有关如何运行示例的说明，请首先查看[函数链接 - Hello 序列示例](durable-functions-sequence.md)。 示例可以在本地运行，也可以发布到 Azure。
8. 在命令提示/终端中运行以下命令以启动项目：
    ```
    func start
    ```

### <a name="start-with-an-empty-project"></a>从空项目开始
 
1. 在命令提示/终端中导航到用于承载函数应用的文件夹。
3. 运行以下命令创建函数应用项目：

    ```bash
    func init
    ``` 
4. 运行 Azure 存储模拟器（仅限 Windows），或使用 `AzureWebJobsStorage` 的实际 Azure 存储连接字符串更新 local.settings.json 文件。
5. 接下来，运行以下命令创建新函数，然后遵循向导中的步骤操作：

    ```bash
    func new
    ```
    >[!IMPORTANT]
    > 目前尚未提供持久函数模板，但可以先从支持的某个选项着手，然后修改代码。 用于引用[业务流程客户端](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx/HttpStart)、[业务流程触发器](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx/E1_HelloSequence)和[活动触发器](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx/E1_HelloSequence)的示例。
2. 在命令提示符/终端窗口中运行下面的命令，以在函数应用目录中安装 Azure Functions 持久扩展：

    ```
    func extensions install
    ```

6. 在 Visual Studio Code 中打开项目文件夹，然后修改模板代码以继续。 
7. 在命令提示/终端中运行以下命令以启动项目：
    ```
    func start
    ```

## <a name="azure-portal"></a>Azure 门户

如果你愿意，可以使用 [Azure 门户](https://portal.azure.com)进行 Durable Functions 开发。

   > [!NOTE]
   > JavaScript 中的 Durable Functions 尚不可在门户中使用。

### <a name="create-an-orchestrator-function"></a>创建一个业务流程协调程序函数

1. 在门户中创建一个新的函数应用，如 [Functions 快速入门文章](functions-create-first-azure-function.md#create-a-function-app)中所示。

2. 对该函数应用进行配置以[使用 2.0 运行时版本](set-runtime-version.md)。

   Durable Functions 扩展对 1.X 运行时和 2.0 运行时都起作用，但是 Azure 门户模板仅在以 2.0 运行时为目标时才可用。

3. 选择“创建自己的自定义函数”创建一个新函数。

4. 将“语言”更改为“C#”，将“方案”更改为“Durable Functions”，选择“Durable Functions Http 初学者 - C#”模板。

5. 在“未安装的扩展”下，单击“安装”来从 NuGet.org 下载扩展。 

6. 安装完成后，选择“Durable Functions Http 初学者 - C#”模板继续创建业务流程客户端函数“HttpStart”。

7. 现在，通过“Durable Functions 业务流程协调程序 - C#”模板创建业务流程函数“HelloSequence”。

8. “Durable Functions 活动 - C#”模板中的最后一个函数名为“Hello”。

9. 转到“HttpStart”函数并复制其 URL。

10. 使用 Postman 或 cURL 调用持久函数。 测试之前，请将 URL 中的 **{functionName}** 替换为业务流程协调程序函数的名称 - **HelloSequence**。  不需要提供数据，只需使用 POST 谓词。 

    ```bash
    curl -X POST https://{your function app name}.azurewebsites.net/api/orchestrators/HelloSequence
    ```

11. 然后，调用“statusQueryGetUri”终结点，随后会看到持久函数的当前状态

    ```json
        {
            "runtimeStatus": "Running",
            "input": null,
            "output": null,
            "createdTime": "2017-12-01T05:37:33Z",
            "lastUpdatedTime": "2017-12-01T05:37:36Z"
        }
    ```

12. 继续调用“statusQueryGetUri”终结点，直到状态更改为“Completed” 

    ```json
    {
            "runtimeStatus": "Completed",
            "input": null,
            "output": [
                "Hello Tokyo!",
                "Hello Seattle!",
                "Hello London!"
            ],
            "createdTime": "2017-12-01T05:38:22Z",
            "lastUpdatedTime": "2017-12-01T05:38:28Z"
        }
    ```

祝贺你！ 现已在 Azure 门户中启动并运行第一个持久函数！

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [运行函数链接示例](durable-functions-sequence.md)
