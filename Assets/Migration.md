### Migration:
This document describes the required steps to successfully migrate from the V1 connector to the V2 Nedap ONS User Connector.
This is still a draft and only validated by the connector development team, and should be validated and updated by the consultancy team before the official release.

## Existing V1 connector
- Check if All actions are successfully Process => So no Errors
- <Force update account + permissions>
- Start migration
- Overwrite all scripts in the connector (Follow migration steps)
  - Also update Configuration.json (There is a new version for the v2 connector) (Example: ApiUrl => BaseUrl)
  - Add the Account Import script
  - Update mapping scripts (New headers for DefaultScope and Roles)
- [Migrate]
- Fix the account Reference
  - Copy ARef fix code to Update.ps1 script (Code is at the bottom of this document)
  - Update all accounts
  - Run enforcement
  - Test permissions (preview) script if Aref is working
  - Make sure to that the reference of the DefaultScope keeps the same, although the V2 version of the Nedap Connector uses the actual reference.
    - Where in the V1 the reference of the DefaultScope permission wasn't used. The reference in the V2 is used, and should be named: 'DefaultScope'

Roles
- Reference Cleaner
  - Remove DisplayName + DisplayNameFull from Permissions script
  - Start **Reference Cleaner**
  - Fix identification
  - Stop Reference Cleaner
 IF NOT FIX =>
    - All Permissions scripts will fail.
    - Subpermissions calculation will fail.
    - Datastorage permissions ID will be incorrectly.
    - Audit Logging will show incorrect permission changes.
    - Account Reference is saved incorrectly.

DefaultSope
- Force update permission to store the permissions in datastorage.

 IF NOT FIX =>
Revoke Defaultscope permisison before "force update" => Defaultscope permissions are not revoked. Because the permissions are not saved in the datastorage.

## Code snippet for possible failsafe when references are not cleaned correctly
FailSave when not removed DisplayName + DisplayNameFull (Subpermissions)
``` powershell
Where-Object { $_.Name -ne 'id' }
```
=>
``` powershell
Ctrl + H => Where-Object { $_.Name -ne 'id' -and $_.name -ne 'DisplayNameFull' -and $_.name -ne 'DisplayName'}
```


### Code sippet to fix Account Reference After migration from V1 to V2 Nedap Ons User Connector
``` powershell
#
if (-not [string]::IsNullOrEmpty($($actionContext.References.Account))) {
    if ($actionContext.References.Account.GetType().fullname -eq 'System.Object[]') {
        $accountReferences = [PSCustomObject]@{}
        foreach ($account in $actionContext.References.Account) {
            $accountReferences | Add-Member @{
                $account.IdentificationNo = @{
                    Uuid             = $account.userUuid
                    IdentificationNo = $account.IdentificationNo
                }
            }
        }
        $actionContext.References.Account = $accountReferences
    }
}
```