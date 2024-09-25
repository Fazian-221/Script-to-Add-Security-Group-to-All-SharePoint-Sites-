# Load SharePoint CSOM Assemblies
Add-Type -Path "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.dll"
Add-Type -Path "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\16\ISAPI\Microsoft.SharePoint.Client.Runtime.dll"

# Initialize variables
$AdminSiteURL = "https://abc-admin.sharepoint.com/"
$SecurityGroup = "Sharepoint-online-Administrators"

# Connect to SharePoint Online Admin Center
Connect-SPOService -Url $AdminSiteURL -Credential $Cred

# Get All Site Collections
$Sites = Get-SPOSite -Limit All

# Function to Add Security Group to a Site
Function Add-SecurityGroupToSite {
    param (
        [string]$SiteURL,
        [string]$GroupName
    )
    # Setup the context
    $Ctx = New-Object Microsoft.SharePoint.Client.ClientContext($SiteURL)
    $Ctx.Credentials = New-Object Microsoft.SharePoint.Client.SharePointOnlineCredentials($Cred.Username, $Cred.Password)
    # Get the web
    $Web = $Ctx.Web
    $Ctx.Load($Web)
    $Ctx.ExecuteQuery()
    # Create or get the security group
    $GroupCreationInfo = New-Object Microsoft.SharePoint.Client.GroupCreationInformation
    $GroupCreationInfo.Title = $GroupName
    $Group = $Web.SiteGroups.Add($GroupCreationInfo)
    $Ctx.Load($Group)
    $Ctx.ExecuteQuery()
    # Add the group to the web's groups collection
    $Web.AssociatedGroups.Add($Group)
    $Ctx.ExecuteQuery()
    Write-Host "Added security group $GroupName to site $SiteURL"
}
# Loop through each site collection and add the security group
ForEach($Site in $Sites) {
    Write-Host -f Yellow "Processing Site: $($Site.Url)"
    Try {
        Add-SecurityGroupToSite -SiteURL $Site.Url -GroupName $SecurityGroup
    }
    Catch {
        Write-Host -f Red "Error adding security group to site $($Site.Url): $_.Exception.Message"
    }
}

# Disconnect from SharePoint Online
Disconnect-SPOService
![Uploading image.png…]()
