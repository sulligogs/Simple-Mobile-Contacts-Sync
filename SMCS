#Simple Mobile Contacts Sync by sulligogs - syncs company users as contacts to their company phones via O365 and Outlook mobile.  v1.0 18/1/2021

param([Parameter(Mandatory)] $ClientID, [Parameter(Mandatory)] $TenantID, [Parameter(Mandatory)] $Thumbprint, $GroupFilter="All Company", $CategoryRemoval)

$Modules=(Get-Module -ListAvailable).Name; "Microsoft.Graph.Users", "Microsoft.Graph.PersonalContacts" | % {if(!($Modules -contains $_)) {Install-Module $_ -Force}}
Connect-Graph -ClientId $ClientID -TenantId $TenantID -CertificateThumbprint $Thumbprint

#Absentees are those in Master Contacts but not personal contacts and orphans in personal contacts but not Master Contacts
$MASTER=0; $PERSONAL=1; $Contacts=@(),@()
$ABSENTEES=0; $ORPHANS=1; $DUPLICATES=2; $RogueContacts=@(),@(),@()

$UserProperties="Mail", "GivenName", "Surname", "JobTitle", "MobilePhone", "BusinessPhones", "StreetAddress", "City", "State", "Country", "PostalCode", "CompanyName", "UserPrincipalName", "AccountEnabled"
$ContactProperties="Id", "EmailAddresses", "GivenName", "Surname", "Jobtitle", "MobilePhone", "BusinessPhones", "CompanyName", "BusinessAddress", "Categories"

while($true)
{
    $Users=Get-MgUser -All -Property $UserProperties | where {((Get-MgUserMember -UserId $_.UserPrincipalName).Id)+"All Company" -contains $GroupFilter -and $_.AccountEnabled} | select $UserProperties
    $Users | ForEach-Object{foreach($Property in $_.PSObject.Properties){if($Property.Value -eq $null){$Property.Value=""}}} #Converts null values to empty strings

    #Prepares a Master Contacts list from current users with same properties as personal contacts
    $Contacts[$MASTER]=@(); $Users | foreach `
    {
        $Contacts[$MASTER]+=
        @{
            EmailAddresses=@(@{Address=$_.Mail; Name="$($_.GivenName) $($_.Surname)"});
            GivenName=$_.GivenName; Surname=$_.Surname;
            JobTitle=$_.JobTitle;
            MobilePhone=$_.MobilePhone; BusinessPhones=@($_.BusinessPhones);
            CompanyName=$_.CompanyName;
            BusinessAddress=@{Street=$_.StreetAddress; City=$_.City;State=$_.State; CountryOrRegion=$_.Country; PostalCode=$_.PostalCode}
        }
    }

    foreach($User in $Users.UserPrincipalName)
    {
        $Contacts[$PERSONAL]=Get-MgUserContact -UserId $User -All -Property $ContactProperties | where {$_.Categories -eq "SMCS"} | select $ContactProperties
        $RogueContacts[$ABSENTEES]=$RogueContacts[$ORPHANS]=$RogueContacts[$DUPLICATES]=@()
        
        #Compares Master Contacts in personal contacts and vice versa
        for($List=$MASTER;$List -le $PERSONAL;$List++)
        {
            $AlternateList=-$List+1
            foreach($SourceContact in $Contacts[$List])
            {
                $ContactMatches=@()
                foreach($TargetContact in $Contacts[$AlternateList])
                {
                    if
                    (
                        $SourceContact.EmailAddresses[0].Address -eq $TargetContact.EmailAddresses[0].Address -and
                        $SourceContact.GivenName -eq $TargetContact.GivenName -and
                        $SourceContact.Surname -eq $TargetContact.Surname -and
                        $SourceContact.JobTitle -eq $TargetContact.JobTitle -and 
                        $SourceContact.MobilePhone -eq $TargetContact.MobilePhone -and 
                        $SourceContact.BusinessPhones[0] -eq $TargetContact.BusinessPhones[0] -and
                        $SourceContact.BusinessAddress.Street -eq $TargetContact.BusinessAddress.Street -and
                        $SourceContact.BusinessAddress.City -eq $TargetContact.BusinessAddress.City -and
                        $SourceContact.BusinessAddress.State -eq $TargetContact.BusinessAddress.State -and
                        $SourceContact.BusinessAddress.CountryOrRegion -eq $TargetContact.BusinessAddress.CountryOrRegion -and
                        $SourceContact.BusinessAddress.PostalCode -eq $TargetContact.BusinessAddress.PostalCode
                    ) 
                    {
                        $ContactMatches+=$TargetContact #Negates an absentee or orphan and records duplicates 
                        if($List -eq $PERSONAL) {break} #Proved not an orphan so move to the next personal contact
                    }
                }

                if($ContactMatches.Count -eq 0) {$RogueContacts[$List]+=$SourceContact} #Accumulates absent or orphaned contacts
                if($ContactMatches.Count -gt 1) {$RogueContacts[$DUPLICATES]+=$ContactMatches[0..($ContactMatches.Count-2)]} #Same with duplicates, but omits original
            }
        }
        
        #Adds any absent personal contacts
        foreach($RogueContactsAbsentee in $RogueContacts[$ABSENTEES]) {New-MgUserContact -UserId $User -Categories "SMCS" @RogueContactsAbsentee}

        #Removes any orphaned personal contacts
        foreach($RogueContactsOrphan in $RogueContacts[$ORPHANS]) {Remove-MgUserContact -UserId $User -ContactId $RogueContactsOrphan.Id}

        #Removes any duplicate personal contacts
        foreach($RogueContactsDuplicate in $RogueContacts[$DUPLICATES]) {Remove-MgUserContact -UserId $User -ContactId $RogueContactsDuplicate.Id}

        #Removes any unwanted personal contacts
        if($CategoryRemoval){Get-MgUserContact -UserId $User -All -Property Id, Categories | where {$_.Categories -eq $CategoryRemoval} | % {Remove-MgUserContact -UserId $User -ContactId $_.Id}}
    }
}  
