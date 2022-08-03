## Storage account

Create storage account as per the last lesson
```
az storage account create  --kind StorageV2   --resource-group learn-cf521fdd-5ca1-4f7e-b1f7-5a8c7e4f33f2   --location centralus   --name 20220729anastest
```

## Container
The app you'll be working with in this module uses a single container. You're going to follow the best practice of letting the app create the container at startup. However, you can create containers from the Azure CLI. If you'd like to see the documentation, run the `az storage container create -h` command in Cloud Shell.

## Deploy and run in azure
Your app is finished, so let's deploy it and see it work. Create an App Service app and configure it with app settings for your storage account connection string and container name. Get the storage account's connection string with az storage account show-connection-string, and set the name of the container to be files.

The app name needs to be globally unique, so you'll need to choose your own name to fill in <your-unique-app-name>. You'll also use the storage account name you created previously to replace <your-unique-storage-account-name>. Run each of the following commands in the below order in Azure CLI:
```
az appservice plan create \
--name blob-exercise-plan \
--resource-group learn-cf521fdd-5ca1-4f7e-b1f7-5a8c7e4f33f2 \
--sku FREE --location centralus
```

```
az webapp create \
--name <your-unique-app-name> \
--plan blob-exercise-plan \
--resource-group learn-cf521fdd-5ca1-4f7e-b1f7-5a8c7e4f33f2


```

```
CONNECTIONSTRING=$(az storage account show-connection-string \
--name <your-unique-storage-account-name> \
--output tsv)


```

```
az webapp config appsettings set \
--name anaskhlapp --resource-group learn-cf521fdd-5ca1-4f7e-b1f7-5a8c7e4f33f2 \
--settings AzureStorageConfig:ConnectionString=$CONNECTIONSTRING AzureStorageConfig:FileContainerName=files
```

```
dotnet publish -o pub
cd pub
zip -r ../site.zip *
```

```
az webapp deployment source config-zip --src ../site.zip --name anaskhlapp --resource-group learn-cf521fdd-5ca1-4f7e-b1f7-5a8c7e4f33f2
```


Try uploading and downloading some files to test the app. After you've uploaded a few files, to see the blobs that have been uploaded to the container, run the following code in the shell, replacing <your-unique-storage-account-name> with the storage account name you created earlier in the module:

```
az storage blob list --account-name <your-unique-storage-account-name> --container-name files --query [].{Name:name} --output table

```

## Further readings
Index data from Azure Blob Storage using Azure Cognitive Search https://docs.microsoft.com/en-us/azure/search/search-howto-indexing-azure-blob-storage
Naming and Referencing Containers, Blobs, and Metadata  https://docs.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata
Security recommendations for Blob storage https://docs.microsoft.com/en-us/azure/storage/blobs/security-recommendations
Configure an ASP.NET Core app for Azure App Service https://docs.microsoft.com/en-us/azure/app-service/configure-language-dotnetcore
Configure a Java app for Azure App Service https://docs.microsoft.com/en-us/azure/app-service/configure-language-java