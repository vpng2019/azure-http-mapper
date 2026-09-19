# Azure HTTP Mapper

An Azure Logic App Standard solution that accepts an XML payload through an HTTP request trigger, transforms the payload using an XSLT map, and returns the transformed XML in the HTTP response.

## Repository

**Repository name:** `azure-http-mapper`

**Suggested GitHub repository description:**

> Azure Logic App Standard HTTP mapper that transforms incoming XML payloads with an XSLT map and returns the mapped XML response, with ARM deployment templates included.

## Solution overview

The solution contains three source/configuration artifacts:

- `workflow/HTTPMapper.json` — Logic App Standard workflow definition.
- `deployment/template.json` — ARM deployment template for the Logic App Standard host resource.
- `deployment/parameters.json` — Deployment parameter values/placeholders.

The workflow is stateful and is triggered by an HTTP request. The request body is passed to the XSLT transformation. After the transformation succeeds, the workflow returns HTTP status `200` and uses the transformed body as the response.

### High level flow

```text
HTTP Client
    |
    |  XML request body
    v
When an HTTP request is received
    |
    v
Transform_XML
    |
    |  XSLT map: Map.xslt
    v
Response (HTTP 200)
    |
    |  transformed XML
    v
HTTP Client
```

## Infrastructure template

`deployment/template.json` provisions an Azure Logic App Standard host using the `Microsoft.Web/sites` resource type.

Key characteristics present in the supplied template include:

- Resource kind: `functionapp,workflowapp`
- System assigned managed identity
- HTTPS only access enabled
- IPv4 networking
- FTPS only publishing
- TLS minimum version `1.2`
- One worker / minimum elastic instance count of `1`
- App Service Plan supplied through the `serverfarms_ASP_RGAPIM_aea9_externalid` parameter
- Hostname binding for `<logic-app-name>.azurewebsites.net`
- FTP and SCM publishing credentials disabled

## Prerequisites

Before deployment, have:

- An Azure subscription
- Permission to create/manage Logic App Standard resources and App Service Plans
- An existing App Service Plan, or a plan you will reference using `serverfarms_ASP_RGAPIM_aea9_externalid`
- The `Map.xslt` mapping file used by the workflow
- Access to GitHub repository `vpng2019/azure-http-mapper` (assuming `vpng2019` is the GitHub owner/account name)

## Deploying the ARM template

### Azure CLI

From the repository root:

```bash
az deployment group create   --resource-group <RESOURCE_GROUP_NAME>   --template-file deployment/template.json   --parameters @deployment/parameters.json   --parameters sites_logicapp_azure_http_mapper_name=logicapp-azure-http-mapper                serverfarms_ASP_RGAPIM_aea9_externalid="/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Web/serverfarms/<APP_SERVICE_PLAN>"
```

Replace the placeholders with values from your Azure environment.

After deployment:

1. Confirm the HTTP request trigger exists.
2. Confirm the `Map.xslt` mapping is available.
3. Send a representative XML payload to the workflow endpoint.
4. Confirm the HTTP response is `200`.
5. Validate that the returned body is the XSLT transformed XML.

## Security and repository hygiene

Do not commit secrets, access keys, SAS tokens, connection strings, client secrets, or other credentials to GitHub.

The provided deployment artifacts contain environment specific Azure resource identifiers. Review `deployment/template.json` before publishing a public repository and replace or parameterize organization specific values as required by your repository's security policy.

## Git commands

Initialize the local repository:

```bash
git init
git branch -M main
git add .
git commit -m "Initial commit - Azure HTTP Mapper"
```

Connect it to GitHub:

```bash
git remote add origin https://github.com/vpng2019/azure-http-mapper.git
git push -u origin main
```

To verify:

```bash
git remote -v
git status
```

## Testing checklist

- [ ] Logic App Standard host deployed
- [ ] Correct App Service Plan referenced
- [ ] HTTP request trigger available
- [ ] `Map.xslt` deployed/available
- [ ] XML request accepted
- [ ] XSLT transformation succeeds
- [ ] HTTP `200` response returned
- [ ] Response body contains the expected transformed XML
- [ ] No credentials or secrets committed to Git
