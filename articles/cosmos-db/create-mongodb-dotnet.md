---
title: Azure Cosmos DB：使用 .NET 和 MongoDB API 生成 Web 应用 | Microsoft Docs
description: 演示一个可以用来连接到 Azure Cosmos DB MongoDB API 并进行查询的 .NET 代码示例
services: cosmos-db
author: slyons
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: quick start connect, mvc
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/22/2018
ms.author: sclyon
ms.openlocfilehash: 7ab02cf2cc9a25a5c4c7aa6d782d37d932dc8369
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701945"
---
# <a name="azure-cosmos-db-build-a-mongodb-api-web-app-with-net-and-the-azure-portal"></a>Azure Cosmos DB：使用 .NET 和 Azure 门户生成 MongoDB API Web 应用

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-golang.md)
>  

Azure Cosmos DB 是 Microsoft 提供的全球分布式多模型数据库服务。 可快速创建和查询文档、键/值和图形数据库，所有这些都受益于 Azure Cosmos DB 核心的全球分布和水平缩放功能。 

本快速入门教程演示如何使用 Azure 门户创建 Azure Cosmos DB [MongoDB API](mongodb-introduction.md) 帐户、文档数据库和集合。 然后生成并部署基于 [MongoDB .NET 驱动程序](https://docs.mongodb.com/ecosystem/drivers/csharp/)构建的“任务列表”Web 应用。

## <a name="prerequisites-to-run-the-sample-app"></a>运行示例应用的先决条件

若要运行该示例，需要 [Visual Studio](https://www.visualstudio.com/downloads/) 和有效的 Azure CosmosDB 帐户。

如果没有 Visual Studio，请在使用安装程序安装 **ASP.NET 和 Web 开发** 工作负荷后，下载 [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

<a id="create-account"></a>
## <a name="create-a-database-account"></a>创建数据库帐户

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

本文中所述的示例与 MongoDB.Driver 版本 2.6.1 兼容。

## <a name="clone-the-sample-app"></a>克隆示例应用

首先，从 GitHub 下载示例 MongoDB API 应用。 这会实施一个包含 MongoDB 文档存储模型的任务列表。

1. 打开命令提示符，新建一个名为“git-samples”的文件夹，然后关闭命令提示符。

    ```bash
    md "C:\git-samples"
    ```

2. 打开诸如 git bash 之类的 git 终端窗口，并使用 `cd` 命令更改为要安装示例应用的新文件夹。

    ```bash
    cd "C:\git-samples"
    ```

3. 运行下列命令以克隆示例存储库。 此命令在计算机上创建示例应用程序的副本。 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started.git
    ```

如果不想使用 git，也可以[下载 ZIP 文件格式的项目](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started/archive/master.zip)。

## <a name="review-the-code"></a>查看代码

此步骤是可选的。 如果有意了解如何使用代码创建数据库资源，可以查看以下代码片段。 否则，可以直接跳转到[更新连接字符串](#update-your-connection-string)。 

以下代码片段全部摘自 DAL 目录中的 Dal.cs 文件。

* 初始化 Mongo 客户端。

    ```cs
        MongoClientSettings settings = new MongoClientSettings();
        settings.Server = new MongoServerAddress(host, 10255);
        settings.UseSsl = true;
        settings.SslSettings = new SslSettings();
        settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

        MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
        MongoIdentityEvidence evidence = new PasswordEvidence(password);

        settings.Credential = new MongoCredential("SCRAM-SHA-1", identity, evidence);

        MongoClient client = new MongoClient(settings);
    ```

* 检索数据库和集合。

    ```cs
    private string dbName = "Tasks";
    private string collectionName = "TasksList";

    var database = client.GetDatabase(dbName);
    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
    ```

* 检索所有文档。

    ```cs
    collection.Find(new BsonDocument()).ToList();
    ```

* 创建任务并将其插入到 MongoDB 集合

   ```csharp
    public void CreateTask(MyTask task)
    {
        var collection = GetTasksCollectionForEdit();
        try
        {
            collection.InsertOne(task);
        }
        catch (MongoCommandException ex)
        {
            string msg = ex.Message;
        }
    }
   ```
   同样，可以使用 [collection.UpdateOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.updateOne/index.html) 和 [collection.DeleteOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.deleteOne/index.html) 方法更新和删除文档。 

## <a name="update-your-connection-string"></a>更新连接字符串

现在返回到 Azure 门户，获取连接字符串信息，并将其复制到应用。

1. 在 [Azure 门户](http://portal.azure.com/)的 Azure Cosmos DB 帐户的左侧导航栏中，单击“连接字符串”，并单击“读写密钥”。 使用屏幕右侧的复制按钮将用户名、密码和主机复制到下一步的 Dal.cs 文件中。

2. 打开 **DAL** 目录中的 **Dal.cs** 文件。 

3. 从门户中复制“用户名”值（使用复制按钮），并在 **Dal.cs** 文件中将其设为“用户名”的值。 

4. 然后从门户复制“主机”值，并在 **Dal.cs** 文件中将其设为“主机”的值。 

5. 最后从门户复制“密码”值，并在 **Dal.cs** 文件中将其设为“密码”的值。 

现已使用与 Azure Cosmos DB 进行通信所需的所有信息更新应用。 
    
## <a name="run-the-web-app"></a>运行 Web 应用

1. 在 Visual Studio 中，右键单击解决方案资源管理器中的项目，并单击“管理 NuGet 包”。 

2. 在 NuGet“浏览”框中，键入 *MongoDB.Driver*。

3. 从结果中安装“MongoDB.Driver”库。 这会安装 MongoDB.Driver 包以及所有依赖项。

4. 单击 Ctrl+F5 运行应用程序。 应用会显示在浏览器中。 

5. 在浏览器中单击“创建”，并在“任务列表”应用中创建一些新任务。

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 门户中查看 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门教程中，已了解如何创建 Azure Cosmos DB 帐户和使用 MongoDB API 运行 Web 应用。 现在可以将其他数据导入 Cosmos DB 帐户。 

> [!div class="nextstepaction"]
> [将 MongoDB API 的数据导入 Azure Cosmos DB](mongodb-migrate.md)

