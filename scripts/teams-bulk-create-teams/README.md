---
plugin: add-to-gallery
---

# Bulk Create Teams with JSON File

## Summary

We had a scenario to manage teams in an environment that we needed to test governance scenarios requiring us to make over a 1000 teams. So we created a script that could generate the required teams, to build a lot of teams without having to generate a long list of team names, we used a prefix series to help build this out, additionally we added to the team name, a suffix to enable us to find all test based teams.

![Example Screenshot](assets/example.png)

Few points to note:

- This application uses an Azure AD app to provision teams with the certificate installed on the executing machine.
- The script requires the JSON block saved as "teams.json" to run.
- The execution time to create each team varies between 5-30 secs per team
- The JSON + Script examples will generate 100 teams.

# [PnP PowerShell](#tab/pnpps)
```powershell

[CmdletBinding()]
param (
    [string] $Tenant = "contoso",
    [string] $SiteUrlSuffix = "-Test",
    [string] $OwnerEmail = "paul.bullock@contoso.co.uk",
    [string] $SiteListJsonFile = ".\teams.json",
    [string] $Template = "STS#3",
    [string] $ClientId = "5fc099aa-d7ad-4da6-a3ca-5793facc5ed3",
    [string] $ThumbPrint = "EE20D40FC1EE7EE5BEC8512617CA469718DA123D"
)
begin{

    $adminUrl = "https://$($Tenant)-admin.sharepoint.com"
    Write-Host "Connecting to SharePoint Admin..."
    $adminConn = Connect-PnPOnline -ClientId $ClientId -Thumbprint $ThumbPrint -Tenant contoso.co.uk -Url $adminUrl -ReturnConnection

    $baseUrl = "https://$($Tenant).sharepoint.com/sites/"
    $jsonFilePath = "$($SiteListJsonFile)"
    $sites = Get-Content $jsonFilePath -Raw | ConvertFrom-Json

    $prefixes = "HR","Finance","ICT","Service Desk","Client Services","Project Alpha","Project Beta","Project Charlie","Leadership","Community"
}
process {

    $prefixes | Foreach-Object{

        $prefix = $_

        $sites |  Foreach-Object {

            $siteUrl = "$($baseUrl)$($_.SiteUrl.Replace("XXXXX", $prefix).Replace(" ",''))$($SiteUrlSuffix)"
            $siteTitle = "$($_.SiteTitle.Replace("XXXXX", $prefix))"
            $mailNickname = "$($_.SiteUrl.Replace("XXXXX", $prefix).Replace(" ",''))$($SiteUrlSuffix)"

            # Check for existing site
            Write-Host "Checking for existing site $($siteUrl)"
            $existingSite = Get-PnPTenantSite $siteUrl -ErrorAction Silent -Connection $adminConn

            if ($existingSite -eq $null) {

                Write-Host " - Creating new Team...." -ForegroundColor Cyan
                New-PnPMicrosoft365Group  -DisplayName $siteTitle -Description "Testing Site for $($siteTitle)" -MailNickname $mailNickname `
                        -Owners $OwnerEmail -IsPrivate:(!$_.IsPublic) -CreateTeam     
                
            }else{
                Write-Host " - Site Exists... Skipping" -ForegroundColor Yellow
            }

        }
    }
}
end{

  Write-Host "Done! :)" -ForegroundColor Green
}
```

[!INCLUDE [More about PnP PowerShell](../../docfx/includes/MORE-PNPPS.md)]

# [JSON](#tab/json)
```json
[
    {"SiteTitle":"XXXXX Collaboration", "SiteUrl":"XXXXX-Collaboration", "IsPublic":true},
    {"SiteTitle":"XXXXX Standards", "SiteUrl":"XXXXX-Standards", "IsPublic":true},
    {"SiteTitle":"XXXXX Published Documents", "SiteUrl":"XXXXX-PublishedDocuments", "IsPublic":false},
    {"SiteTitle":"XXXXX Client Services", "SiteUrl":"XXXXX-ClientServices", "IsPublic":false},
    {"SiteTitle":"XXXXX Management Team", "SiteUrl":"XXXXX-ManagementTeam", "IsPublic":false},
    {"SiteTitle":"XXXXX National Rollout", "SiteUrl":"XXXXX-NationalRollout", "IsPublic":false},
    {"SiteTitle":"XXXXX Internal Review and Audit", "SiteUrl":"XXXXX-InternalReviewAndAudit", "IsPublic":false},
    {"SiteTitle":"XXXXX Technical Services", "SiteUrl":"XXXXX-TechnicalServices", "IsPublic":false},
    {"SiteTitle":"XXXXX Planning and Performance", "SiteUrl":"XXXXX-PlanningAndPerformance", "IsPublic":false},
    {"SiteTitle":"XXXXX Operations", "SiteUrl":"XXXXX-Operations", "IsPublic":false}
]
```

> Note: the XXXXX will be replaced as part of the script

***


## Contributors

| Author(s) |
|-----------|
| Paul Bullock |


[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://telemetry.sharepointpnp.com/script-samples/scripts/template-script-submission" aria-hidden="true" />