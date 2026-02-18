---
title: "Azure API Management (APIM)"
description: "Azure APIM CLI commands and policy snippets for API gateway configuration, security, and rate limiting."
category: "Azure APIM"
---

## Service Management (CLI)

```bash
# Create an APIM instance
az apim create \
  --name myAPIM \
  --resource-group myRG \
  --publisher-email admin@example.com \
  --publisher-name "My Org" \
  --sku-name Consumption

# List APIM instances
az apim list -o table

# Import API from OpenAPI spec
az apim api import \
  --resource-group myRG \
  --service-name myAPIM \
  --api-id my-api \
  --path /myapi \
  --specification-format OpenApiJson \
  --specification-url "https://example.com/swagger.json"

# List APIs
az apim api list -g myRG -n myAPIM -o table
```

## Common Policies

### Rate Limiting

```xml
<inbound>
    <!-- 100 calls per 60 seconds per subscription -->
    <rate-limit calls="100" renewal-period="60" />

    <!-- OR quota: 10000 calls per 7 days -->
    <quota calls="10000" renewal-period="604800" />
</inbound>
```

### JWT Validation

```xml
<inbound>
    <validate-jwt header-name="Authorization" require-scheme="Bearer"
                  failed-validation-httpcode="401"
                  failed-validation-error-message="Unauthorized">
        <openid-config url="https://login.microsoftonline.com/{tenant-id}/v2.0/.well-known/openid-configuration" />
        <audiences>
            <audience>api://my-api-client-id</audience>
        </audiences>
        <issuers>
            <issuer>https://sts.windows.net/{tenant-id}/</issuer>
        </issuers>
        <required-claims>
            <claim name="roles" match="any">
                <value>Api.Read</value>
                <value>Api.Write</value>
            </claim>
        </required-claims>
    </validate-jwt>
</inbound>
```

### CORS

```xml
<inbound>
    <cors allow-credentials="true">
        <allowed-origins>
            <origin>https://myapp.example.com</origin>
        </allowed-origins>
        <allowed-methods preflight-result-max-age="300">
            <method>GET</method>
            <method>POST</method>
            <method>PUT</method>
            <method>DELETE</method>
            <method>OPTIONS</method>
        </allowed-methods>
        <allowed-headers>
            <header>Authorization</header>
            <header>Content-Type</header>
        </allowed-headers>
    </cors>
</inbound>
```

### Caching

```xml
<inbound>
    <cache-lookup vary-by-developer="false"
                  vary-by-developer-groups="false"
                  downstream-caching-type="none">
        <vary-by-query-parameter>version</vary-by-query-parameter>
    </cache-lookup>
</inbound>
<outbound>
    <cache-store duration="3600" />
</outbound>
```

### Request/Response Transformation

```xml
<inbound>
    <!-- Add header -->
    <set-header name="X-Request-ID" exists-action="skip">
        <value>@(context.RequestId.ToString())</value>
    </set-header>

    <!-- Rewrite URL -->
    <rewrite-uri template="/api/v2/{path}" />
</inbound>
<outbound>
    <!-- Remove internal headers -->
    <set-header name="X-Powered-By" exists-action="delete" />
    <set-header name="X-AspNet-Version" exists-action="delete" />
</outbound>
```

### Backend Circuit Breaker

```xml
<backend>
    <retry condition="@(context.Response.StatusCode >= 500)"
           count="3" interval="1" first-fast-retry="true">
        <forward-request buffer-request-body="true" timeout="30" />
    </retry>
</backend>
```
