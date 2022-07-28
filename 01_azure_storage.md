## Create an Azure storage account

We'll use the `az storage account create` command to create a new storage account. There are several parameters to control the configuration of the storage account.

```
az storage account create \
  --resource-group learn-bfa5c621-5477-4f60-8c50-5f8806a5c094 \
  --location westus \
  --sku Standard_LRS \
  --name <name>
```

## Interact with the Azure Storage APIs

Azure Storage provides a REST API to work with the containers and data stored in each account. Each type of data you can store has its own independent API. Recall that we have four specific data types:

- Blobs for unstructured data such as binary and text files.
- Queues for persistent messaging.
- Tables for structured storage of key/values.
- Files for traditional SMB file shares.


## Create a .NET Core application
```
dotnet new console --name PhotoSharingApp
cd PhotoSharingApp
dotnet run
```

## Use the REST API
```GET https://[url-for-service-account]/?comp=list&include=metadata```


Results are 
```XML
<?xml version="1.0" encoding="utf-8"?>  
<EnumerationResults AccountName="https://[url-for-service-account]/">  
  <Containers>  
    <Container>  
      <Name>container1</Name>  
      <Url>https://[url-for-service-account]/container1</Url>  
      <Properties>  
        <Last-Modified>Sun, 24 Sep 2018 18:09:03 GMT</Last-Modified>  
        <Etag>0x8CAE7D0C4AF4487</Etag>  
      </Properties>  
      <Metadata>  
        <Color>orange</Color>  
        <ContainerNumber>01</ContainerNumber>  
        <SomeMetadataName>SomeMetadataValue</SomeMetadataName>  
      </Metadata>  
    </Container>  
    <Container>  
      <Name>container2</Name>  
      <Url>https://[url-for-service-account]/container2</Url>  
      <Properties>  
        <Last-Modified>Sun, 24 Sep 2018 17:26:40 GMT</Last-Modified>  
        <Etag>0x8CAE7CAD8C24928</Etag>  
      </Properties>  
      <Metadata>  
        <Color>pink</Color>  
        <ContainerNumber>02</ContainerNumber>  
        <SomeMetadataName>SomeMetadataValue</SomeMetadataName>  
      </Metadata>  
    </Container>  
    <Container>  
      <Name>container3</Name>  
      <Url>https://[url-for-service-account]/container3</Url>  
      <Properties>  
        <Last-Modified>Sun, 24 Sep 2018 17:26:40 GMT</Last-Modified>  
        <Etag>0x8CAE7CAD8EAC0BB</Etag>  
      </Properties>  
      <Metadata>  
        <Color>brown</Color>  
        <ContainerNumber>03</ContainerNumber>  
        <SomeMetadataName>SomeMetadataValue</SomeMetadataName>  
      </Metadata>  
    </Container>  
  </Containers>  
  <NextMarker>container4</NextMarker>  
</EnumerationResults>  
```

## Use client library

```C#
string containerName = "...";
BlobContainerClient container = new BlobContainerClient(connectionString, containerName);

var blobs = container.GetBlobs();
foreach (var blob in blobs)
{
    Console.WriteLine($"{blob.Name} --> Created On: {blob.Properties.CreatedOn:YYYY-MM-dd HH:mm:ss}  Size: {blob.Properties.ContentLength}");
}
```

##  Add the storage client library to your app (Nuget)

```
dotnet add package Azure.Storage.Blobs
dotnet run
```


## Connect to your Azure storage account
You have added the required client libraries to your application and are ready to connect to your Azure storage account.
To work with data in a storage account, your app will need two pieces of data:

- Access key
- REST API endpoint

Each storage account requores access key and to connect we need the endpoints. The followng are the endpoints:

- Blobs: https://[name].blob.core.windows.net/
- Queues	https://[name].queue.core.windows.net/
- Table	https://[name].table.core.windows.net/
- Files	https://[name].file.core.windows.net/

## Storage account connection string

```
DefaultEndpointsProtocol=https;AccountName={your-storage};
   AccountKey={your-access-key};
   EndpointSuffix=core.windows.net
```

Commoon praactice isn't to commit your keys to the repository or store it in configuration files. Rotating keys is a best practice. 

## Add Azure Storage configuration to your app
1- Create a JSON configuration file 

```
cd PhotoSharingApp
touch appsettings.json
code . --to open the code editor
```

```JSON
{
    "ConnectionStrings": {
        "StorageAccount": "<value>"
    }
}
```

## Obtain actual storage account 
```
az storage account show-connection-string \
  --resource-group learn-bfa5c621-5477-4f60-8c50-5f8806a5c094 \
  --query connectionString \
  --name <name>
```
This command will return the connection string, you can use it to connect.

Add the following to the csproj
```XML
<ItemGroup>
    <None Update="appsettings.json">
        <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
</ItemGroup>
```
## Add nuget to read json
```
dotnet add package Microsoft.Extensions.Configuration.Json
```

Your code:
```C#
using System;
using Microsoft.Extensions.Configuration;
using System.IO;
using Azure.Storage.Blobs;


namespace PhotoSharingApp
{
    class Program
    {
        static void Main(string[] args)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json");

            var configuration = builder.Build();

            var connectionString = configuration.GetConnectionString("StorageAccount");
            string containerName = "photos";
            BlobContainerClient container = new BlobContainerClient(connectionString, containerName);
            container.CreateIfNotExists();

          // Uploads the image to Blob storage.  If a blb already exists with this name it will be overwritten
            string blobName = "docs-and-friends-selfie-stick";
            string fileName = "docs-and-friends-selfie-stick.png";
            BlobClient blobClient = container.GetBlobClient(blobName);
            blobClient.Upload(fileName, true);

          // List out all the blobs in the container
            var blobs = container.GetBlobs();
            foreach (var blob in blobs)
            {
                Console.WriteLine($"{blob.Name} --> Created On: {blob.Properties.CreatedOn:yyyy-MM-dd HH:mm:ss}  Size: {blob.Properties.ContentLength}");
            }
        }
    }
}
```
```
dotnet run
Thgis command is to show the container created above
az storage container list \
--account-name <name>
```
