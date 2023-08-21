[Docker command cheatsheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)


Defining multiple docker arguments
In Dockerfile, we can define multiple build arguments with multiple ARG commands like

ARG VERSION
ARG PORT
and in docker build command we have to use multiple — build-arg

docker build -t hello-world:latest --build-arg VERSION=0.2 --build-arg PORT=80 .


[Dockerfile Reference](https://docs.docker.com/engine/reference/builder/#entrypoint)

[The Compose file](https://docs.docker.com/compose/compose-file/03-compose-file/)


### ACR
Reference: [az acr](https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest)

- Create an Azure Container Registry
    ```sh
    az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Basic
    ```
    > SKU: `Basic`, `Standard`, `Premium`, `Classic`

- Get ACR list
    ```sh
    az acr list -o table
    ```
- Get ACR Detail 
    ```sh
    az acr show -n $ACR_NAME -g $RESOURCE_GROUP
    # Get only ACR ID
    az acr show -n $ACR_NAME -g $RESOURCE_GROUP --query "id" -o tsv
    ```
- Show ACR Repositories
    ```sh
    # Show list of repositories
    az acr repository list -n $ACR_NAME -o table

    Result
    ----------------
    azure-vote-back
    azure-vote-front
    testcontainer
    food-recognition
    web-front

    # Show the detail of a repository
    az acr repository show  -n $ACR_NAME --repository $REPO_NAME -o table

    CreatedTime                   ImageName     LastUpdateTime                ManifestCount    Registry               TagCount
    ----------------------------  ------------  ----------------------------  ---------------  ---------------------  ----------
    2019-01-17T05:19:36.6227367Z  captureorder  2019-04-05T04:50:34.8244574Z  5                myazconacr.azurecr.io  5

    # Show list of tags in a repository
    az acr repository show-tags -n $ACR_NAME --repository $REPO_NAME -o table

    Result
    --------
    21
    32
    55
    56
    59

    ```
- Login to ACR 
    ```sh
    az acr login --name $ACR_NAME

    # Alternatively login with docker command
    ACR_LOGIN_SERVER=$ACR_NAME.azurecr.io
    docker login $ACR_LKOGIN_SERVER -u $ACR_USER -p $ACR_PASSWORD
    ```
- ACR Task - Build
    >  You can queues a quick build, providing streamed logs for an Azure Container Registry by using [az acr build](https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest#az-acr-build)

    ```sh
    az acr build --registry $ACR_NAME --image [CONTAINER_NAME:TAG] [SOURCE_LOCATION]

    ## More usages are:
    #Queue a local context (folder), pushed to ACR when complete, with streaming logs.
    az acr build -t sample/hello-world:{{.Run.ID}} -r MyRegistry .

    # Queue a local context, pushed to ACR without streaming logs.
    az acr build -t sample/hello-world:{{.Run.ID}} -r MyRegistry --no-logs .

    # Queue a local context to validate a build is successful, without pushing to the registry using the --no-push parameter.
    az acr build -t sample/hello-world:{{.Run.ID}} -r MyRegistry --no-push .

    # Queue a local context to validate a build is successful, without pushing to the registry. Removing the -t parameter defaults to --no-push
    az acr build -r MyRegistry .
    ```

[ACR Security Guildlines](https://learn.microsoft.com/en-us/security/benchmark/azure/baselines/container-registry-security-baseline)
