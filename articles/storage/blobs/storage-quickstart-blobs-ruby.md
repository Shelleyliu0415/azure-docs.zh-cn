---
title: "Azure 快速入门 - 使用 Ruby 将对象转移到 Azure Blob 存储或从 Azure Blob 存储转移对象 | Microsoft Docs"
description: "快速了解如何使用 Ruby 将对象转移到 Azure Blob 存储或从 Azure Blob 存储转移对象"
services: storage
author: ruthogunnnaike
manager: jeconnoc
ms.service: storage
ms.tgt_pltfrm: na
ms.devlang: ruby
ms.topic: quickstart
ms.date: 12/7/2017
ms.author: v-ruogun
ms.openlocfilehash: 30cda1997ca93f237dafd34cfffc2a86b86d6c7c
ms.sourcegitcommit: fa28ca091317eba4e55cef17766e72475bdd4c96
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/14/2017
---
#  <a name="transfer-objects-tofrom-azure-blob-storage-using-ruby"></a>使用 Ruby 将对象转移到 Azure Blob 存储或从 Azure Blob 存储转移对象
本快速入门介绍如何使用 Ruby 上传、下载和列出 Azure Blob 存储的容器中的块 Blob。 

## <a name="prerequisites"></a>先决条件

完成本快速入门教程： 
* 安装 [Ruby](https://www.ruby-lang.org/en/downloads/)
* 使用 rubygem 包安装[用于 Ruby 的Azure 存储库](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-ruby-how-to-use-blob-storage#configure-your-application-to-access-storage)。 

```
gem install azure-storage
```

如果还没有 Azure 订阅，可以在开始前创建一个 [免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [storage-quickstart-tutorial-create-account-portal](../../../includes/storage-quickstart-tutorial-create-account-portal.md)]

## <a name="download-the-sample-application"></a>下载示例应用程序
本快速入门中使用的[示例应用程序](https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git)是基本的 Ruby 应用程序。  

使用 [git](https://git-scm.com/) 可将应用程序的副本下载到开发环境。 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-ruby-quickstart.git 
```

此命令会将存储库克隆到本地 git 文件夹。 若要打开 Ruby 示例应用程序，请查找 storage-blobs-ruby-quickstart 文件夹，然后打开 example.rb 文件。  

## <a name="configure-your-storage-connection-string"></a>配置存储连接字符串
在应用程序中，必须提供存储帐户名称和帐户密钥，以创建应用程序的 `Client` 实例。 从 IDE 中的解决方案资源管理器打开 `example.rb` 文件。 将 accountname 和 accountkey 值分别替换为帐户名称和密钥。 

```ruby 
client = Azure::Storage.client(storage_account_name: account_name, storage_access_key: account_key)
```

## <a name="run-the-sample"></a>运行示例
此示例将在“文档”文件夹中创建一个测试文件。 示例程序会将该测试文件上传到 Blob 存储，列出容器中的 blob，并使用新名称下载此文件。 

运行示例。 以下输出是运行应用程序时返回的输出的示例：
  
```
Temp file = C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

Uploading to Blob storage as blobQuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

List blobs in the container
         Blob name: QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

Downloading blob to C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078_DOWNLOADED.txt
```
按任意键继续时，示例程序会删除存储容器和文件。 继续前，请在“文档”文件夹中查看这两个文件。 可以打开它们，并看到它们完全相同。

还可以使用工具（如 [Azure 存储资源管理器](http://storageexplorer.com)）查看 Blob 存储中的文件。 Azure 存储资源管理器是免费的跨平台工具，可用于访问存储帐户信息。 

验证文件后，按任意键可完成演示并删除测试文件。 了解此示例的用途以后，即可打开 example.rb 文件来查看代码。 

## <a name="understand-the-sample-code"></a>了解示例代码

接下来逐步介绍示例代码，以便展示其工作方式。

### <a name="get-references-to-the-storage-objects"></a>获取对存储对象的引用
首先创建对用于访问和管理 Blob 存储的对象的引用。 这些对象相互关联，并且每个对象被列表中的下一个对象使用。

* 创建 Azure 存储**客户端**对象的实例，用于设置连接凭据。 
* 创建 **BlobService** 对象，使之指向存储帐户中的 Blob 服务。 
* 创建**容器**对象，让其代表要访问的容器。 容器用于组织 blob，就像使用计算机上的文件夹组织文件一样。

有了云 Blob 容器后，可以创建**块** Blob 对象，使之指向你感兴趣的特定 Blob，然后执行上传、下载、复制等操作。

> [!IMPORTANT]
> 容器名称必须为小写。 有关容器名称和 blob 名称的详细信息，请参阅[命名和引用容器、Blob 和元数据](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)。

此部分设置 Azure 存储客户端的实例，实例化 Blob 服务对象，创建新的容器，然后设置容器的权限，使 Blob 公开。 容器名称为 quickstartblobs。 

```ruby 
# Setup a specific instance of an Azure::Storage::Client
client = Azure::Storage.client(storage_account_name: account_name, storage_access_key: account_key)

# Create the BlobService that represents the Blob service for the storage account
blob_service = client.blob_client

# Create a container called 'quickstartblobs'.
container_name ='quickstartblobs'
container = blob_service.create_container(container_name)   

# Set the permission so the blobs are public.
blob_service.set_container_acl(container_name, "container")
```

### <a name="upload-blobs-to-the-container"></a>将 blob 上传到容器

Blob 存储支持块 blob、追加 blob 和页 blob。 块 blob 是最常用的 blob，此快速入门中使用的便是它。  

若要将文件上传到 blob，请通过将本地驱动器上的目录名称和文件名称联接在一起来获取文件的完整路径。 然后可以使用 **create\_block\_blob()** 方法将文件上传到指定的路径。 

示例代码将创建一个本地文件，以供上传和下载，并将要上传的此文件存储为 file\_path\_to\_file，将 blob 的名称存储为 local\_file\_name。 以下示例将文件上传到名为“quickstartblobs”的容器。

```ruby
# Create a file in Documents to test the upload and download.
local_path=File.expand_path("~/Documents")
local_file_name ="QuickStart_" + SecureRandom.uuid + ".txt"
full_path_to_file =File.join(local_path, local_file_name)

# Write text to the file.
file = File.open(full_path_to_file,  'w')
file.write("Hello, World!")
file.close()

puts "Temp file = " + full_path_to_file
puts "\nUploading to Blob storage as blob" + local_file_name

# Upload the created file, using local_file_name for the blob name
blob_service.create_block_blob(container.name, local_file_name, full_path_to_file)
```

若要对块 Blob 进行部分更新，请使用 **create\_block\_list()** 方法。 块 blob 最大可以为 4.7 TB，并且可以是从 Excel 电子表格到大视频文件的任何内容。 页 blob 主要用于用于备份 IaaS VM 的 VHD 文件。 追加 blob 用于日志记录，例如有时需要写入到文件，再继续添加更多信息。 追加 Blob 应在单编写器模型中使用。 存储在 Blob 存储中的大多数对象都是块 blob。

### <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

可以使用 **list\_blobs()** 方法获取容器中文件的列表。 下面的代码检索 blob 列表，然后循环访问它们，显示在容器中找到的 blob 的名称。  

```ruby
# List the blobs in the container
puts "\n List blobs in the container"
blobs = blob_service.list_blobs(container_name)
blobs.each do |blob|
    puts "\t Blob name #{blob.name}"   
end  
```

### <a name="download-the-blobs"></a>下载 Blob

使用 **get\_blob()** 方法将 Blob 下载到本地磁盘。 以下代码将下载前面部分所上传的 blob。 会将“_DOWNLOADED”添加为 blob 名称的前缀，以便可以在本地磁盘上同时看到这两个文件。 

```ruby
# Download the blob(s).
# Add '_DOWNLOADED' as prefix to '.txt' so you can see both files in Documents.
full_path_to_file2 = File.join(local_path, local_file_name.gsub('.txt', '_DOWNLOADED.txt'))

puts "\n Downloading blob to " + full_path_to_file2
blob, content = blob_service.get_blob(container_name,local_file_name)
File.open(full_path_to_file2,"wb") {|f| f.write(content)}
```

### <a name="clean-up-resources"></a>清理资源
如果不再需要本快速入门中上传的 Blob，可使用 **delete\_container()** 方法删除整个容器。 如果不再需要已创建的文件，请使用 **delete\_blob()** 方法将文件删除。

```ruby
# Clean up resources. This includes the container and the temp files
blob_service.delete_container(container_name)
File.delete(full_path_to_file)
File.delete(full_path_to_file2)    
```

## <a name="next-steps"></a>后续步骤
 
本快速入门介绍了如何使用 Ruby 在本地磁盘和 Azure Blob 存储之间转移文件。 要深入了解如何使用 Blob 存储，请继续学习 Blob 存储操作说明。

> [!div class="nextstepaction"]
> [Blob 存储操作说明](./storage-ruby-how-to-use-blob-storage.md)


若要详细了解存储资源管理器和 Blob，请参阅[使用存储资源管理器管理 Azure Blob 存储资源](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。