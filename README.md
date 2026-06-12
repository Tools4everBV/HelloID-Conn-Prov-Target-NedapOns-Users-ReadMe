# HelloID-Conn-Prov-Target-NedapOns-Users

> [!WARNING] Work in Progress
> This connector is still a work in progress, and its documentation and code may change without prior notice. It is created and validated by the development team and will be thoroughly tested by our consultancy team before the official release. The transition to the V2 connector impacts the existing V1 connector and requires several extensive migration steps.

> [!IMPORTANT]
> This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.

> [!WARNING]
> Extensive knowledge. This connector requires the existence of Nedap Employees inside Nedap Ons. Limited Nedap employee support can be achieved using [this connector](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Employee-Readme).
Extensive knowledge of HelloID provisioning and Nedap Ons (Nedap user and Nedap employee) are required.

> [!WARNING]
> Upgrade warning! Since the last update in (November) 2024/2025, Nedap has altered the authorization structure, which has had an impact on the API. The connector has been adjusted to integrate these changes in the API while also providing backward compatibility support. For further details, refer to chapters `Backwards Compatible` and `DefaultScope` in the Public README.md.


<p align="center">
  <img src="https://www.tools4ever.nl/assets/connectors/helloid-conn-prov-target-nedapons-users-readme.png">
</p>


## Table of contents

- [HelloID-Conn-Prov-Target-NedapOns-Users](#helloid-conn-prov-target-nedapons-users)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Supported features](#supported-features)
  - [Getting started](#getting-started)
    - [HelloID Icon URL](#helloid-icon-url)
    - [Requirements](#requirements)
    - [Connection settings](#connection-settings)
    - [Correlation configuration](#correlation-configuration)
    - [Available lifecycle actions](#available-lifecycle-actions)
    - [Field mapping](#field-mapping)
    - [Script Mapping](#script-mapping)
      - [Script Mapping lookup values](#script-mapping-lookup-values)
  - [Remarks](#remarks)
    - [Generic](#generic)
      - [Connector Scope](#connector-scope)
      - [DataStorage](#datastorage)
      - [Unmanage removed entitlement(s)](#unmanage-removed-entitlements)
      - [Single Agent](#single-agent)
      - [Preview Mode (dryRun):](#preview-mode-dryrun)
      - [Account Reference](#account-reference)
      - [MappingFiles](#mappingfiles)
    - [Multiple accounts](#multiple-accounts)
      - [Processing Multiple Accounts](#processing-multiple-accounts)
      - [Account reference Validation Check](#account-reference-validation-check)
      - [SubPermissions](#subpermissions)
    - [Authorization](#authorization)
      - [Scope Settings (Advanced Scope)](#scope-settings-advanced-scope)
      - [DefaultScope (legacy)](#defaultscope-legacy)
      - [Backwards Compatible](#backwards-compatible)
      - [CSV Lookup](#csv-lookup)
    - [Notifications](#notifications)
      - [IsCreated | IsDeleted](#iscreated--isdeleted)
      - [Example Configuration: ](#example-configuration-)
  - [Known Issues](#known-issues)
    - [Remove a Nedap roles from DataStorage](#remove-a-nedap-roles-from-datastorage)
    - [Account Reference Conversion](#account-reference-conversion)
    - [Permission DisplayName (v1 only)](#permission-displayname-v1-only)
  - [Governance Remarks](#governance-remarks)
    - [Import](#import)
      - [Already linked accounts](#already-linked-accounts)
      - [EmployeeNumber](#employeenumber)
      - [Import configuration](#import-configuration)
      - [Account access](#account-access)
    - [Reconciliation](#reconciliation)
      - [Delete action](#delete-action)
      - [Notification](#notification)
      - [Account loop](#account-loop)
  - [Provisioning](#provisioning)
    - [Roles Permissions](#roles-permissions)
    - [DefaultScope Permissions](#defaultscope-permissions)
    - [Supported Properties](#supported-properties)
  - [Fact Sheet](#fact-sheet)
  - [Development resources](#development-resources)
    - [API endpoints](#api-endpoints)
    - [API documentation](#api-documentation)
  - [Getting help](#getting-help)
  - [HelloID docs](#helloid-docs)

## Introduction
This Repository does only contain the README. The source code can be found in a private repository and is meant only for internal use. Link to the repository: [Nedap Ons Users](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users)

Nedap Ons provides a REST API to programmatically interact with its services and data. The connector manages the Nedap accounts, DefaultScope, and Provisioning roles. The roles and the DefaultScope can be assigned as entitlement and the scope of the teams and locations are calculated based on a specified property in the HelloID contracts. To map the property to the actual Nedap Team or Location additional mapping is required.

## Supported features

The following features are available:

| Feature                                          | Supported | Actions                                                                                                  | Remarks           |
| ------------------------------------------------ | --------- | -------------------------------------------------------------------------------------------------------- | ----------------- |
| **Account Lifecycle**                            | ✅         | Create, Update, Delete (Disable)                                                                         |                   |
| **Permissions Roles**                            | ✅         | Retrieve, Grant, Revoke                                                                                  | Dynamic           |
| **Permissions DefaultScope**                     | ✅         | Retrieve, Grant, Revoke                                                                                  | Static or Dynamic |
| **Resources**                                    | ❌         | Used to create mapping validation files                                                                  |                   |
| **Entitlement Import: Accounts**                 | ✅         | [Import remarks](#import)                                                                                |                   |
| **Entitlement Import: Permissions Roles**        | ❌         | No retrieve available.                                                                                   |                   |
| **Entitlement Import: Permissions DefaultScope** | ❌         | No retrieve available.                                                                                   |                   |
| **Governance Reconciliation Resolutions**        | ✅⚠️       | Governance reconciliation is supported for reporting purposes. [Governance Remarks](#governance-remarks) |                   |

## Getting started

### HelloID Icon URL
URL of the icon used for the HelloID Provisioning target system.
```
https://www.tools4ever.nl/assets/connectors/helloid-conn-prov-target-nedapons-users-readme.png
```

### Requirements
- **Direct HR Employees Sync**: <br>
 Direct HR employees synchronization with Nedap to manage the employees in Nedap.
- **HelloID DataStorage**: <br>
 The HelloID DataStorage must be enabled.
- **SSL Certificate**: <br>
  A valid Nedap certificate (.PFX) *Tools4ever needs to request a certificate from Nedap to access the REST API*
- **Mapping Files**: <br>
   - Mapping between HR departments to Nedap clients/locations for determining the scope for the Nedap Provisioning roles and possibly for the DefaultScope.
   - Mapping between HR teams to Nedap team/employee for determining the scope for the Nedap Provisioning roles and possibly for the DefaultScope.
 - **Limit Scope possibilities**: <br>
  Determine the scope **Types** that are required for the role assignments. The connector supports default **thirteen scope possibilities**. The overview can be overwhelming to the customer in the Business Rules overview. This means that there are thirteen entitlements created per Nedap Role. Please remove the entitlement types which not apply to your needs, by removing the code in the entitlement script.<br>
- **New Implementation**<br>
  You can remove the Configuration item: `Grant 'Myself' to DefaultScope (Legacy)` and the DefaultScope entitlement: `DefaultScope (legacy)` <br>
- **Custom HelloId Property**: <br>
  A custom property on the HelloID Person contract with a combination of the employeeCode and EmploymentCode called: [custom.NedapOnsIdentificationNo]
Example:
  ```javascript
  function getValue() {
      return sourceContract.PersonCode + "-" + sourceContract.EmploymentCode
  }
  getValue();
  ```

### Connection settings

The following settings are required to connect to the API.

| Setting                                | Description                                                                                                 |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Environment URL API                    | Development, Acceptance, Production                                                                         |
| Certificate (.PFX) Path                | Full path to Certificate > Nedap-cert.pfx                                                                   |
| Certificate Password                   | Password of the certificate                                                                                 |
| Grant 'Myself' to DefaultScope (Legacy) | When enabled, the scope 'De medewerker zelf' will be granted to the DefaultScope of the account. **Only Used for the DefaultScope (Legacy) Entitlement**             |
| Grant 'Myself' to each Role assignment | When enabled, the scope 'De medewerker zelf' will granted for each Role assignment                           |
| Mapping (Locations)                    | The Path to the mapping file (HR Location => Nedap location 1:M) *Example can be found in the asset folder* |
| Mapping (Teams)                        | The Path to the mapping file (HR Teams =>  Nedap Teams 1:M)  *Example can be found in the asset folder*     |
| CSV Delimiter                          | Mapping File CSV Separation Character                                                                       |
| Explicit Mapping                       | When Enabled, rows that consist of explicit mappings involving both the department and title are not accumulated with rows that solely contain department-related information  |
| Validate Team and Location             | Enable validation of mapped locations and teams                                                             |
| Directory Cache Locations Teams        | Cache directory for current Nedap Ons locations and current Nedap Ons teams                                 |
| Import Only Active Employees.          | When toggled, The Import script will only import user accounts that have active contracts on the employee account in the Nedap Ons system. |
| Days before start of the contract.     | Days before start of the contract. Only used when the `Import Only Active Employees` is enabled.            |
| Days after end of the contract.        | Days after end of the contract. Only used when the `Import Only Active Employees` is enabled.               |

### Correlation configuration

The correlation configuration is used to specify which properties will be used to match an existing account within _NedapOns-User_ to a person in _HelloID_.

| Setting                   | Value                    |
| ------------------------- | ------------------------ |
| Enable correlation        | `True`                   |
| Person correlation field  | `ExternalId`             |
| Account correlation field | `_outputInfo.externalId` |

> [!IMPORTANT]
> Correlation should be enabled even when no direct person properties are used for correlation in the account Life Cycle. When correlation is not enabled, a correlated account will not be updated during account correlation. The Update action will not be triggered when Correlation is disabled. [Correlation configuration](#correlation-configuration).
>
> The correlation properties are solely for the import and reconciliation. [See remark EmployeeNumber](#employeenumber).

> [!TIP]
> _For more information on correlation, please refer to our correlation [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems/correlation.html) pages_.


### Available lifecycle actions

The following lifecycle actions are available:

| Action                          | Description                                                                                                                                                 |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create.ps1                      | Creates or Correlates multiple users account in the target system for each Employment (employeeId + SequenceNumber), based on the contracts in condition from the Business Rules. |
| Update.ps1                      | Creates, (updates), or deletes Account references                                                                                                           |
| Delete.ps1                      | Removes existing account reference(s)                                                                                                                       |
| Roles\permissions.ps1           | Get Nedap Roles, with 13 options [Roles Permissions](#roles-permissions)                                                                                                             |
| Roles\subPermissions.ps1        | Grant/Update/Revoke Nedap Provisioning Roles                                                                                                                |
| Defaultscope\permissions.ps1    | Static values, you can use the defaultScopeEntitlements script or enter six static entitlements like [DefaultScope Permissions](#defaultscope-permissions) |
| Defaultscope\subPermissions.ps1 | Grant/Update/Revoke DefaultScope (myTeams, myLocations, and Scope settings).                                                                                |
| Resource.ps1                    | Create validation files to check against the given Nedap location a team Ids in the mapping files                                                           |
| import.ps1                      | Import the existing account from Nedap *(Configurable by active)*                                                                                          |
| configuration.json              | Contains the connection settings and general configuration for the connector.                                                                               |
| fieldMapping.json               | Defines mappings between person fields and target system person account fields.                                                                             |


### Field mapping

The field mapping can be imported by using the _fieldMapping.json_ file.

### Script Mapping
Besides the configuration and field mapping, you can also configure script variables to decide which property from the HelloID contracts is used to look up a value in the mapping tables and how the primary contract calculation should be done. Please note that some `same` configuration must be applied in both the create and update scripts, as shown below:

#### Script Mapping lookup values
```Powershell
# lookup Value of the person object
$teamPrimaryLookupKey =       { $_.Department.ExternalId }                # Mandatory
$teamSecondaryLookupKey =     { $_.Title.ExternalId }                   # Not mandatory
$locationPrimaryLookupKey =   { $_.Department.ExternalId }            # Mandatory
$locationSecondaryLookupKey = { $_.Title.ExternalId }               # Not mandatory
```
> [!NOTE].
These mapping can be found in the default scope and role permission scripts.

## Remarks
### Generic
#### Connector Scope
  This connector only manages the users and the authorizations. And is intended to be used along with a direct sync HR. AFAS, for example. So the Employee objects are not managed in this connector. The connector depends on this sync. When an employee object is not found the user cannot be created.

#### DataStorage
  The connector uses DataStorage to keep track of the current permissions; `Provisioning Roles` and `DefaultScope`. The DataStorage is behind a feature flag so must be enabled before it can be used in your tenant.


#### Unmanage removed entitlement(s)
**Important:** The Unmanage action in HelloID is not supported for entitlements in this connector.

Nedap permissions are stored in the DataStorage. Unmanaging an entitlement does not remove it from the DataStorage or from Nedap, which may cause unexpected behavior.

Always Revoke the entitlements from the business rule instead of unmanaging it.

#### Single Agent
  Since this connector is using DataStorage, all actions are executed one at a time. Therefore our best practice is the usage of one HelloID Agent for this connector. Also accessing the required local certificate file and CSV mapping files might result in slower processing and/or file locks.

#### Preview Mode (dryRun):
Note that in preview mode (DryRun), all HelloID contracts of a Person are in scope. Therefore, it does not simulate the actual outcome when it comes to determining which account or permissions should be created, updated, or deleted. However, this DryRun mode is added to verify if the mapping, configuration setting, etc. are present and correct. The contracts in scope are normally configured in the business rules. This cannot be stimulated in Preview.

#### Account Reference
The account reference is populated with a hash table containing properties of multiple accounts; the `Uuid` and `IdentificationNo` properties are stored for each *NedapOns-user* account.

#### MappingFiles
The mapping files are used for both role assignments and the DefaultScope in the permission scripts. It is assumed that the application between HR and Nedap is the same. The mapping are used to determine the teams and locations, explicitly for the calculated role assignment or the teams and locations in the DefaultScope.

> [!IMPORTANT]
> It is recommended to execute 'Force Update Permissions' after any modification to the mapping to enforce the desired changes directly in Nedap, as there is no automatic trigger.


### Multiple accounts
#### Processing Multiple Accounts

Due to the support for multiple accounts within Nedap, the Update task may result in the removal of an account. This scenario presents a problem, as the default process order for revoking a trigger is to first revoke the permissions and then revoke the account entitlement. As a result, permissions are revoked before the account entitlement is outside of scope. This process is described in the HelloID documentation. However, in our particular scenario, the process operates differently. The update task first removes the account, resulting in the process order being reversed, with the account revocation occurring before the permission is revoked. This difference in process order leads to the removed account reference not appearing in the permission task, making it impossible to remove the associated permissions. The permission script subsequently performs a cleanup process to revoke the permissions of the previously removed accounts during the next run. However, this is not a straightforward process and will only be triggered during the next specific permission update or when manually prompted to update the permissions.

#### Account reference Validation Check
> *Known error: *No HelloID Account reference(s) found!**
>
In certain situations, an employment with the reference number 1000467-1 may have an account entitlement, while another employment with the reference number 1000467-2 has been granted permissions for the DefaultScope or Provisioning Role. This leads to a mismatch between the account reference and the contracts in scope. The mismatch results from an incorrect configuration of the Business Rules. The connector checks for this mismatch and generates an error and an audit log. Unfortunately, there is a second use case where an account reference cannot be checked beforehand. When an account is not created correctly in the update script, the permission script triggers after 24 hours to update the permissions because the processing order is not forced. *(Read more: [HelloId Processing order](https://docs.helloid.com/en/provisioning/enforcement.html))*
As a result, the permissions script keeps failing until the account is created in the second use case, or in the first use case when the business rules are modified to match the requirements. After this, the problem will be automatically resolved in the next scheduled/manual enforcement.
A drawback of this processing is that the account with a correctly filled account reference gets updated in Nedap Ons. However, because the HelloId action failed, it keeps retrying until the issue with the other account is resolved, also causing the 'correct' account to receive updates each time. Additionally, the desired permissions are not saved in the data storage. When another permission is triggered from HelloID, it gets overwritten because the desired permission is not stored in the data storage. The same problem exists if the business rule has been configured incorrectly.

> [!IMPORTANT]
> To get a closing solution, you can specify the account and permission entitlements in distinct business rules. Additionally, it is suggested to configure the permission entitlement to be out of scope before the account entitlement during off-boarding or re-boarding procedures... To prevent out-of-sync permissions.

#### SubPermissions
Each entitlement shows its own sub-permissions. Because multiple entitlements can provide the same type of access due to backward-compatible entitlements, situations may arise where the sub-permissions displayed in HelloID are not updated to the latest status. For example, if you have the DefaultScope permissions and later add AllEmployees, the SubPermissions of the DefaultScope entitlement cannot be updated during the grant of the AllEmployees entitlement, so it retains its current state. This can be confusing; however, the permissions in Nedap are updated as expected. To 'update' the sub-permissions of the entitlement, you can initiate a 'Force Update' action from the entitlement menu.


### Authorization
#### Scope Settings (Advanced Scope)
- Nedap has introduced a new endpoint specifically to manage the Advanced Scope within the Nedap system, following recent authorization changes. (This was formerly known as the DefaultScope.) This scope is **only** used by other applications within the Nedap application landscape, such as Cockpit.
- These scope settings cannot be applied within the DefaultScope along with role assignments.
- The API does **NOT** overwrite the current values of the Advanced Scope in Nedap. Therefore, when HelloID revokes the scope, it reverts to its original values.

#### DefaultScope (legacy)
The DefaultScope consists of six entitlements, with the 'DefaultScope (legacy)' entitlement existing solely for backward compatibility. This entitlement is a combination of five others: MyLocations, MyTeams, AllClients, AllEmployees and Myself *(where AllClients and AllEmployees are looked up from the CSV file and Myself will be assigned based on connector configuration)*.

**For DefaultScope Permissions:**
Granting these individual entitlements results in the same access as the DefaultScope entitlement, and they can be used together for backward compatibility.

**For Role Assignments:**
The same applies to the default scope in role assignments. The DefaultScope assignment is also present for backward compatibility. Therefore, the recommended practice is to use the more defined entitlements: *DefaultScopeTeams, DefaultScopeLocations, AllClients* and *AllEmployees*.

> [!NOTE]
> **Only use DefaultScope (legacy) in an update scenario.** For new implementations, use the individual entitlements directly.

#### Backwards Compatible
Nedap has introduced a new authorization policy that requires scopes on roles to be assigned more specifically. This primarily means that `AllClients`, `AllEmployees` and `Myself` can no longer be assigned as scopes through the DefaultScope. These must now be assigned directly to the role.
To prevent requiring our customers to immediately switch to the new authorization model, we have adjusted the connector so that the transition is not yet enforced.

Now, when assigning a role with the DefaultScope, the connector checks the existing mapping file to see if the contract of the person originally had access to `AllClients` or `AllEmployees`. If this is the case, we will include the role's scope with `AllClients` or `AllEmployees`.

> [!CAUTION]
> This backward compatibility feature is required only for existing implementations. For new implementations, you should avoid using these entitlements and instead start directly with the new individual entitlements (DefaultScopeTeams, DefaultScopeLocations, AllClients, AllEmployees, Myself) to meet Nedap's requirements.

#### CSV Lookup
- The entitlement `Permission - DefaultScope (legacy)` performs a lookup in the mapping to retrieve AllEmployees and AllClients.
- The entitlements `Permission - :RoleName - Type: DefaultScoped` also perform a lookup in the mapping to retrieve AllEmployees and AllClients.
- Permission scopes granted from the mapping are marked with a `(csv)` in the audit logs and in the SubPermissions, indicating that the scope originates from the mapping file and not directly from an entitlement.

### Notifications
#### IsCreated | IsDeleted
The connector has two properties: `IsCreated` and `IsDeleted`. These properties are used for custom notifications. The connector cannot use the standard notification because there are multiple accounts per person. Therefore, it is possible that an account is created in the Update script, or during creation, one of the two accounts is correlated, triggering an account update. This means there will be no standard creation trigger

The properties are populated with the accounts that are created or deleted, respectively, in the Create, Update, and Delete actions. When there are no creations or deletions, it returns  `null`. Therefore, you will need multiple custom events to cover all cases. You can also use the `Has Value` filter. Example notification message: *Created accounts with the following NedapOnsIdentificationNo's: [{{DATA._outputInfo.isCreated}}]*

#### Example Configuration: <br>
**Custom Events:**
 <img src="assets/Custom Events.png">

**Custom Event Configuration:**
 <img src="assets/Custom Events Configuration.png">

**Notification Configurations:**
 <img src="assets/Notification Configurations.png">

> [!TIP]
> Read more about [Custom Notification](https://docs.helloid.com/en/provisioning/notifications--provisioning-/custom-notification-events--conditional-notifications-.html).


## Known Issues
### Remove a Nedap roles from DataStorage
Known Issue: Unable to remove a Nedap role from the Data Storage after it has been deleted in Nedap. [Known Issue: Remove Nedap Role DataStorage](https://forum.helloid.com/forum/helloid-connectors/general/4988-known-issue-a-removed-nedap-role-saved-in-the-helloid-datastroage)

### Account Reference Conversion
To support Nedap Account import scripts, the account reference object has been changed to a HashTable. Previously, the account reference was stored as an array. If you already have a Nedap User V2 version running, you can copy the code block below to the top of the update script. After that, click "Update all Accounts" and run an Enforcement. This will convert the "old" account references to the current format.
Once the process is complete, you may remove the code block. However, leaving it in the update script will not cause any issues.

```PowerShell
# Fix to migrate from the old AccountReferences to the new AccountReference.
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

### Permission DisplayName (v1 only)
 The display names of the permissions in HelloID are cached, and they only refresh after a specific time limit has been reached. As a result, the display name of the permissions is not directly saved in HelloID and therefore, not in the PowerShell scripts. Previous versions of the system, before December 28, 2022, relied on this display name. However, this dependency has been removed. Unfortunately, previously granted permissions will not be automatically corrected with the new display name and will continue to rely on the old display name. To avoid any issues caused by this, you can implement the following code as a temporary fix until all the granted permissions are re-granted.
  ```Powershell
  if ('DisplayName' -notin $pRef.PSObject.Properties.name  ) {
      if ($eRef.PermissionDisplayName -ne '<unknown permission>') {
          $pRef | Add-Member -NotePropertyMembers @{
              DisplayName = "$($eRef.PermissionDisplayName.Split('-')[1].trim(' '))"
          }
      }
  }
  ```

## Governance Remarks
The Nedap connector supports importing Nedap employee accounts, and the import functionality can be used as normal. However, there are some important remarks to keep in mind, see the  [Import](#import) remarks section for details. Reconciliation is intended for **reporting** purposes only.

### Import
#### Already linked accounts
The import only detects users without an existing linked Nedap account in HelloID. Users who already have a linked account and receive additional Nedap accounts won’t appear in the entitlement import. Therefore, the import script is primarily intended for initial implementation.

#### EmployeeNumber
To perform person correlation, the IdentificationNo is split on the dash (-) to extract the employee number. This differs from the approach used in the Account Lifecycle, where a custom property combining the employee number and contract number is used to correlate individual accounts.

To store the employee number, the `_outputInfo.externalId` property is used in the field mapping. Please note that this field is only used by the import script.

#### Import configuration
The import scripts include configuration options for which accounts to retrieve. By default, all accounts are retrieved and shown in the import overview, but you may want to exclude past accounts. This can be done by toggling `ImportOnlyActiveEmployees`, which retrieves only accounts with active contracts.

The `ImportOnlyActiveEmployees` toggle also enables additional settings to expand the range of active contracts considered, using `daysBeforeContractStartDate` and `daysAfterContractEndDate`. These values can be adjusted in the configuration.


#### Account access
Account access is not used in the Nedap Employee Connector; therefore, no `Account Access` entitlements will be granted.

### Reconciliation
Reconciliation for Nedap cannot be fully used because there is a one-to-many relationship between a HelloID person and Nedap accounts. Due to the risk of unwanted account deletions, reconciliation should be used only as a reporting tool to compare HelloID against Nedap. The deletion action is blocked, but creating accounts is still possible—although this is not yet fully supported.

#### Delete action
However, if you try to delete the unmanaged account, HelloID executes the delete action from the account lifecycle, which deletes the account entitlement and consequently removes all accounts from HelloID. Normally, account deletions involving multiple accounts are handled in the Update.ps1 script to manage this situation.
Therefore, deletion from reconciliation is blocked in the delete action to prevent accidental account deletions.

#### Notification
Reconciliation does not support custom or built-in events when deleting accounts through reconciliation.

#### Account loop
Due to multiple accounts per HelloID person, old (unmanaged) accounts cause managed persons in HelloID to repeatedly appear in each report because of mismatches between accounts managed by HelloID and those still existing in Nedap. You cannot exclude the person entirely since they are still in scope, but only some of their accounts are not. Additionally, excluding accounts at this level is not possible.

To minimize this issue, you are able to filter in the import script to retrieve only active accounts. See [Import configuration](#import-configuration)

## Provisioning
Using this connector you will have the ability to create and manage the following items in Nedap:


### Roles Permissions
*	List Nedap Provisioning Roles (Name + GUID)
* Entitlement options: *(Please keep only the scopes the customer need)*
    * DefaultScope (legacy) *Only use in Update scenario*
    * RoleScoped
    * Custom Scope
      * Clients
        * All Clients
        * Clients on my Roster
        * Clients on my Planning
        * No Clients
        * Calculated Clients based on Contracts *(External Mapping required)*
        * DefaultScopedLocations
      * Teams
        * All Teams
        * No Teams
        * My Roster
        * Calculated Teams based on Contracts *(External Mapping required)*
        * DefaultScopedTeams

### DefaultScope Permissions
- Static values: you can use the `defaultScopeEntitlements.ps1` script or enter six static entitlements. These values are then used in the permissions (Defaultscope). The preferable way is to use the script to avoid typos. Make sure that the "references" match the following values.
   - DefaultScope (legacy) *Only use in Update scenario*
   - DefaultScopeTeams
   - DefaultScopeLocations
   - DefaultScopeAllClients
   - DefaultScopeAllEmployees
   - DefaultScopeMyself

> [!WARNING]
> Switching between static and script values results in the loss of entitlements from the configured business rules.

### Supported Properties
| PropertyName            | Notes                                                  |
| ----------------------- | ------------------------------------------------------ |
| UserName                | Mapped as employee number + Employment Sequence Number |
| contractRequiredAtLogin |                                                        |
| SsoEnabled              |                                                        |
| limitLocationView       |                                                        |
| passwordChange          |                                                        |

## Fact Sheet
The following table displays an overview of the functionality of the Nedap Ons connector for HelloID Provisioning and Service Automation.

| Nedap Accounts        | Supported by Nedap                                                                 | Supported by HelloID provisioning | Supported by HelloID Service Automation |
| --------------------- | ---------------------------------------------------------------------------------- | --------------------------------- | --------------------------------------- |
| Create Accounts       | Yes                                                                                | Yes                               | No                                      |
| Update Accounts       | Yes                                                                                | Yes                               | No                                      |
| Delete Accounts       | Yes                                                                                | No, not applicable                | No                                      |
| Disable Accounts      | No                                                                                 | No                                | No                                      |
| Set initial Password  | No                                                                                 | No                                | No                                      |
| Password Reset        | No, *This works only if the account was created in Nedap. Due to a bug in the API* | No                                | No                                      |
| Set Dashboard profiel | No                                                                                 | No                                | No                                      |
<br/>

| Nedap Authorizations                                                              | Supported by  Nedap                                         | Supported by  HelloID provisioning                                                                                                                                                                                                 | Supported by HelloID Service Automation |
| --------------------------------------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Set a user's DefaultScope *(MyTeams and MyLocations)* (standaard bereik)          | Yes                                                         | :warning: Yes, We strongly advise you to contact Tools4ever first before using this feature, *Additional mapping is required.*  <br> 	:warning: This feature requires an additional endpoint relative to existing implementations! | No                                      |
| Assign role with custom scope (aangepast bereik)                                  | Yes                                                         | Yes                                                                                                                                                                                                                                | No                                      |
| Set default scope (standaard bereik) (myTeams and myLocations) in role assignment | Yes                                                         | :warning: Yes, We strongly advise you to contact Tools4ever first before using this feature                                                                                                                                        | No                                      |
| Set role scope (rol bereik) in role assignment                                    | Yes                                                         | Yes                                                                                                                                                                                                                                | No                                      |
| Set custom Locations (Clienten) scope in role assignment                          | Yes                                                         | Yes, using a custom scope, my roster, and my planning. *Additional mapping required*                                                                                                                                               | No                                      |
| Set custom Teams (Medewerkers) scope in role assignment                           | Yes                                                         | Yes, using a custom scope. *Additional mapping required*                                                                                                                                                                           | No                                      |
| Set duration of scope (ValidFrom / ValidTo)                                       | No, *This should be managed in HelloID with business rules* | No                   | No |


## Development resources

### API endpoints

The following endpoints are used by the connector

| Endpoint                                                            | Description                            |
| ------------------------------------------------------------------- | -------------------------------------- |
| /v0/administration/users                                            | Create users                           |
| /v0/administration/users/by_uuid/:accountUuid                       | Retrieve specific user information     |
| /v0/administration/users/by_employee_id/:employeeId                 | Retrieve specific user information     |
| /v0/administration/employees/by_identification_no/:identificationNo | Retrieve specific employee information |
| /v0/xstream/employees/data                                          | Retrieve employees information list    |
| /v0/xstream/responsibilities/data                                   | Retrieve users information list        |
| /v0/authorization/provisioning/users/:accountUuid/my_teams          | Update DefaultScope Permissions        |
| /v0/authorization/provisioning/users/:accountUuid/my_locations      | Update DefaultScope Permissions        |
| /v0/authorization/provisioning/users/:accountUuid/scope_settings    | Update DefaultScope Permissions        |
| /v0/authorization/provisioning/users/:accountUuid/duties            | Update Roles Permissions               |
| /v0/authorization/provisioning/roles                                | Retrieve Role information              |
| /v0/administration/locations                                        | Retrieve Location information          |
| /v0/administration/teams                                            | Retrieve Team information              |

### API documentation
* Nedap API Documentation → [Click](https://www.ons-api.nl/english/technical/APIS.html)
* Nedap Ons Authorization manual → [Click](https://www.ons-api.nl/english/authorization/AuthorizationInOns.html)

## Getting help

> [!TIP]
> _For more information on how to configure a HelloID PowerShell connector, please refer to our [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems.html) pages_.

> [!TIP]
>  _If you need help, feel free to ask questions on our [forum](https://forum.helloid.com/forum/helloid-connectors/provisioning/313-helloid-prov-target-nedap-ons-users)_.

## HelloID docs

The official HelloID documentation can be found at: https://docs.helloid.com/
