```pwsh
$clientId = "44447795-4c24-4a4d-b5d1-0166b0731594"
$clientSecret = "***"
$tenantId = "d38b2db8-335d-45d6-830d-670d6f8ab263"

$tokenBody = @{
    Grant_Type    = "client_credentials"
    Scope         = "https://graph.microsoft.com/.default"
    Client_Id     = $clientId
    Client_Secret = $clientSecret
}

$tokenResponse = Invoke-RestMethod -Uri "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token" -Method POST -Body $tokenBody
$accessToken = $tokenResponse.access_token

$secureAccessToken = ConvertTo-SecureString $accessToken -AsPlainText -Force

Connect-MgGraph -AccessToken $secureAccessToken

$context = Get-MgContext
$context.Scopes

Connect-MgGraph -Scopes  "Policy.ReadWrite.ApplicationConfiguration", "Policy.Read.All", "Application.ReadWrite.All"

$params = @{
    Definition            = @('{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"4:00:00"}}') 
    DisplayName           = "WebPolicyScenario"
    IsOrganizationDefault = $false
}
$tokenLifetimePolicyId = (New-MgPolicyTokenLifetimePolicy -BodyParameter $params).Id

Get-MgPolicyTokenLifetimePolicy -TokenLifetimePolicyId $tokenLifetimePolicyId

$params = @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/policies/tokenLifetimePolicies/$tokenLifetimePolicyId"
}

$applicationObjectId = "b84ec7a1-91c5-4d3b-8d59-9000af774b3c"

New-MgApplicationTokenLifetimePolicyByRef -ApplicationId $applicationObjectId -BodyParameter $params
```

```
Get-MgApplicationTokenLifetimePolicy -ApplicationId $applicationObjectId
Definition                                                              DeletedDateTime Description DisplayName       Id                                   IsOrganizationDefault
----------                                                              --------------- ----------- -----------       --                                   ---------------------
{{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"0:10:00"}}}                             WebPolicyScenario 3494db13-acfd-41be-8d24-289b4309b7b8 False

```
