# HelloID-Conn-Prov-Target-NedapONS-Users-ReadMe

| :information_source: Information |
|:---------------------------|
| This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.       |

<br />

> :warning: **_Information_**   <br />This connector requires the existence of Nedap Employees inside Nedap Ons. Limited Nedap employee support can be achieved using [this connector](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapONS-Employee-Readme).
<br />Extensive knowledge of HelloID provisioning and Nedap Ons (Nedap user and Nedap employee) are required.

<br />
<br />

<p align="center">
  <img src="https://user-images.githubusercontent.com/68013812/94918899-c672c700-04b3-11eb-9132-7125bbf77fa5.png" width=250>
</p>

## Versioning
|  | Description | Date |
| - | - | - |
|   | Added Support for managing DefaultScope <br>  Added Resource.ps1 validation script | 2022-06-15 |
|  | Added Role assigments with role and Default Scope| 2022-03-24 |
|   | Initial release | 2021-08-27 |

<!-- TABLE OF CONTENTS -->
## Table of Contents
* [Versioning](#Versioning)
* [Introduction](#introduction)
* [Getting Started](#getting-started)
* [Connection settings](#Connection-settings)
* [Prerequisites](#prerequisites)
* [Remarks](#Remarks)
* [Provisioning](#provisioning)
  * [Create](#create)
  * [Update](#update)
  * [Delete](#delte)
  * [GetPermissions](#getPermissions)
  * [Grand](#grand)
  * [Permissions Grant/Update/Revoke](#Permissions-Grant/Update/Revoke)
  * [Supported Properties](#Supported-Properties)
* [Fact Sheet](#Fact-Sheet)
  * [Remote Nedap documentatie](#Remote-Nedap-documentatie)
* [Setup the connector](#Setup-The-Connector)
* [HelloID Docs](#helloid-docs)
* [Forum Thread](#forum-thread)


## Introduction
This Repository does only contain the readme. The source code can be found in a private repository and is meant only for internal use. Link to repository: [Nedap Ons Users](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapONS-Users)

Nedap Ons provides a REST API to programmatically interact with its services and data.
The connector manages the Nedap account and Provisioning roles. The roles can be assigned as entitlement and the scope of the teams and locations are calculated based on property in the HelloID contracts. To map the property to actual Nedap Teams or Locations additional mapping is required.


## Getting Started

### Connection settings

The following settings are required to connect to the API.

| Setting     | Description |
| ------------ | ----------- |
| Environment URL API     |    https://api-staging.ons.io                                     |
| Certificate (.PFX) Path    |  Full path to Certificate> Nedap-cert.pfx                       |
| Certificate Password |    Password of the certificate                                       |
| Mapping File (Locations)|  The Path to the mapping file (HR Location => Nedap location 1:M) *Example can be found in the asset folder* |
| Mapping File (Teams)|  The Path to the mapping file (HR Teams =>  Nedap Teams 1:M)  *Example can be found in the asset folder*     |
| Directory Cache Locations Teams|  Cache directory for current Nedap Ons locations and current Nedap Ons teams      |
| CSV Delimiter| Mapping File CSV Separation Character         |
| Validate Team and Location|  Enable validation of mapped locations and teams       |

### Prerequisites

- Direct HR employees synchronization with Nedap to manage the employees in Nedap
- A valid Nedap Certificate (Tools4ever need to requests a certificate by Nedap to access the API)
- Mapping between HR departments to Nedap Clients/Locations for determining the scope for the Nedap Provisioning roles and the possible DefaultScope.
- Mapping between HR Teams to Nedap Team/Employee for determining the scope for the Nedap Provisioning roles and the possible DefaultScope.
- Determine the scope types that are required for the role assignments. The connector supports default **ten scope possibilities**. The overview can be overwhelming to the customer in the entitlement overview. This means that there are ten entitlements created per Nedap Role. Please remove the entitlement types which not apply to your needs, by removing the code in the entitlement script.
- The HelloID DataStorage must be enabled
- An custom property on the HelloID contract with a combination of the employeeCode and EmploymentCode named: [custom.NedapOnsIdentificationNo]
Example:
  ```javascript
  function getValue() {
      return sourceContract.PersonCode + "-" + sourceContract.EmploymentCode
  }
  getValue();
  ```


### Remarks

 - This connector does only manages the users and the authorizations. And is intended to be used along with a direct sync HR. AFAS for example. So the Employee objects are not managed in this connector. The connector depends on this sync. When an employee object is not found the user cannot be created.
 - The connector uses DataStorage to keep track of the current permissions. The DataStorage is behind a feature flag so must be enabled before it can be used in your tenant.
 - Please be careful when changing/adding the permission types. Removing a permission type is not a problem, but adding them (again) might encounter unexpected behavior. The displayname of the permissions are cached in HelloID, and they only refresh after reaching a certain time limit. This cache means that the displayname of the permissions aren't directly saved in HelloID, and likewise not in the PowerShell scripts. The Permissions script relies on the PermissionsDisplayname and cannot process the permissions without the displayname. The entitlements are shown in the entitlement HelloID overview, but cannot be used until the names are properly cached.
 - Also, be careful when enabling to manage the default scope of a User. Because Nedap does not support a get or a patch method to update the default scope of a user. So the scope which is specified in HelloID will overwrite any current default scope of the existing user. And any manual adjustment will be overwritten as soon a update action taken place.
- The mapping files are used for both the role assignments in the permission script and the Default scope. Supposing the mapping applied between HR en Nedap is the same. Although for the Default scope extra columns are added AllEmployee and AllClients.
These columns are ignored in the role assignments. The All options for the role assignments are managed with separated entitlements.


### Provisioning
Using this connector you will have the ability to create and manage the following items in Nedap:


| Files       | Description                                             |
| ----------- | ------------------------------------------              |
| Create.ps1  | Creates or Correlates the user in the target system <br> Set / Set and Update the DefaultScope  |
| Update.ps1  | Creates, update or deletes Account references <br> Set / Set and Update and remove the DefaultScope        |
| Delete.ps1  |  Removes account reference(s)    _(Success = True)_  <br> Removes the DefaultScope    |
| Permission.ps1 | Grant/Update/Revoke Nedap Roles                      |
| Entitlements.ps1  |  Get Nedap Roles, with 10 options _(See below)_ |
| Resource.ps1  | Create validation files to check against the given Nedap location an team Ids in the mapping files |


### Create:

* Multiple user accounts for each unique combination (employeeId + contact sequence number), based on the contracts in condition from the Business Rules
* DefaultScope
  * Set and Update defaultScope foreach account

  Result:
  * One entitlement “Nedap Account”
  * A list of account references that can be used in the account lifecycle.
  *	Audit Logs for each account created.
  * Audit Logs for each DefaultScope what is applied




### Update:
* Update does not make changes, It's not required
* Create a new user account for each new unique (employeeId + contact sequence number) combination.
* Delete Remove account reference from Aref  -*See delete action* -
* DefaultScope
  * Set defaultScope with a new account
  * Update the defaultScope on exisint accounts
  * Remove the defaultScope with a account deletion

  Result:
    * Audit Logs for each account (Create, Update, Delete)
    *	Update the account reference with the new situation.
    * Audit Logs for each DefaultScope what is applied
    *	The entitlement overview should be the same.


### Delete:
*  Remove account reference from AccountReferences
*  Remove the defaultScope
  Result:
    * Remove the Entitlement "Nedap Account"
    *	Remove the Account Reference in HelloID
    *	Audit Logs for each account deleted
    * Audit Logs for each DefaultScope what is removed

### Permissions
*	List Nedap Provisioning Roles (Name + GUID)
* Entitlement options: *(Please keep only the scopes the customer need)*
    * Custom Scope
      * Clients
        * All Clients
        * Clients on my Roster
        * Clients on my Planning
        * No Clients
        * Calculated Clients based on Contracts (External Mapping is needed)
      * Teams
        * All Teams
        * No Teams
        * Calculated Teams based on Contracts (External Mapping is needed)
    * DefaultScoped
    * RoleScoped

### Permissions Grant/Update/Revoke

*(Sequenced after Account lifecycle)*
  * Calculate the desired permissions as a sum of the current Permissions/Entitlements plus the new permission and assign all the Nedap roles at once.

    Result
    * Permission for each entitlement
    *	Audit Logs for each entitlement with a summary of the Scope (Location and Teams) in the Nedap Role.
    * SubPermissions for each entitlement with an Account and location/team combination


### Supported Properties
| PropertyName | Notes |
| ------------ | ------------|
| ContractRequiredAtLogin |  |
| SsoEnabled |  |
| PasswordChange |  |
| UserName | Mapped as employee number + Employment Sequence Number|



___________

## Fact Sheet
The following table displays an overview of the functionality for the Nedap Ons connector for HelloID Provisioning and Service Automation.

|Nedap Accounts |Supported by Nedap    |Supported by HelloID provisioning |Supported by HelloID Service Automation|
| ------------ | ----------- |----------- |----------- |
| Create Accounts|Yes|Yes|No
| Update Accounts  |Yes|Yes|No
| Delete Accounts |Yes|No, not applicable|No
| Disable Accounts |No|No|No
| Set initial Password |No|No|No
| Password Reset |No, *This works only if the account was created in Nedap. Due to a bug in the API*  |No|No
|Set Dashboard profiel |No|No|No
<br/>


|Nedap Authorizations |Supported by  Nedap    |Supported by  HelloID provisioning |Supported by HelloID Service Automation|
| ------------ | ----------- |----------- |----------- |
| Set a user's DefaultScope (standaard bereik) |Yes| :warning: Yes, We strongly advise you to contact Tools4ever first before using this feature, *Additional mapping required.*  <br> 	:warning: This feature requires an additional endpoint relative to existing implementations! |No
| Assign role with custom scope (aangepast bereik)|Yes |Yes |No
| Assign role with default scope (standaard bereik)|Yes |:warning: Yes, We strongly advise you to contact Tools4ever first before using this feature|No
| Assign role with role scope (rol bereik)|Yes |Yes |No
| Set custom Locations (Clienten) scope in role assignment|Yes|Yes, using a custom scope, my roster and my planning. *Additional mapping required*|No
| Set custom Teams (Medewerkers) scope in role assignment|Yes|Yes, using a custom scope. *Additional mapping required*|No
| Set duration of scope (ValidFrom / ValidTo) |No, *This should be managed in HelloID with business rules*|No|No



### Remote Nedap documentation
* Nedap API Documentation → [klik](https://www.ons-api.nl/APIS.html)
* Nedap ONS Authorization manual → [klik](https://ons-api.nl/support/Shield.html)


## Setup the connector

* Before using this connector make sure you enter the configuration and replace the following variables.
 <img src="Assets/configuration.png">

* Besides the configuration tab, you can also configure script variables. To decide which property from a HelloID contract is used to look up a value in the mapping table, this is known as the HR Location or HR Team. And you can configure the Defaultscope behavior. Please note that some "same" configuration must taken place in multiple scripts. Shown as below:

#### Permissions.ps1

  ```PowerShell
  $TeamProperty1              = { $_.Department.ExternalId }  # Mandatory
  $TeamProperty2              = { $_.Title.ExternalId }  # Not mandatory
  $locationProperty1          = { $_.Department.ExternalId }   # Mandatory
  $locationProperty2          = $null # { $_.Title.ExternalId }  # Not used
  $employmentContractFilter   = { $_.Custom.NedapOnsIdentificationNo } #Dienstverband
  ```
#### Create.ps1

  ```PowerShell
$teamLookupField1             = { $_.Department.ExternalId } # Mandatory
$teamLookupField2             = { $_.Title.ExternalId } # Not mandatory
$locationLookupField1         = { $_.Department.ExternalId } # Mandatory
$locationLookupField2         = $null  # { $_.Title.ExternalId }  # Not used
$employmentContractFilter     = { $_.Custom.NedapOnsIdentificationNo }  # Dienstverband

 # Set to true if accounts in the target system must be updated
$updatePerson = $true

 # DefaultScope
    # 'Dont'      # Dont apply
    # 'Once'      # Apply once on new account creation
    # 'Manage'    # Manage DefaultScope, Apply on new and existing accounts
$applyDefaultScope = 'Dont
  ```

#### Update.ps1

  ```PowerShell
$teamLookupField1             = { $_.Department.ExternalId } # Mandatory
$teamLookupField2             = { $_.Title.ExternalId } # Not mandatory
$locationLookupField1         = { $_.Department.ExternalId } # Mandatory
$locationLookupField2         = $null  # { $_.Title.ExternalId }  # Not used
$employmentContractFilter     = { $_.Custom.NedapOnsIdentificationNo }  # Dienstverband
 # DefaultScope
    # 'Dont'      # Dont apply
    # 'Once'      # Apply once on new account creation
    # 'Manage'    # Manage DefaultScope, Apply on new and existing accounts
$applyDefaultScope = 'Dont'

 # Clear the Defaultscope with an account deletion.
    # $false       # Remove the DefaultScope
    # $true        # Do not remove the DefaultScope
$removeDefaultScope = $false

  ```

#### Delete.ps1

  ```PowerShell
 # Clear the Defaultscope with an account deletion.
    # $false       # Remove the DefaultScope
    # $true        # Do not remove the DefaultScope
$removeDefaultScope = $false
  ```



* Configure the Permission definition as follows:
<img src="Assets/PermissionsDefenition.png">

_For more information about our HelloID PowerShell connectors, please refer to our general [Documentation](https://docs.helloid.com/hc/en-us/articles/360012558020-How-to-configure-a-custom-PowerShell-target-connector) page_

## HelloID Docs

The official HelloID documentation can be found at: https://docs.helloid.com/

## Forum Thread
The Forum thread for any questions or remarks regarding this connector can be found at: [Helloid-prov-target-nedap-ons-users](https://forum.helloid.com/forum/helloid-connectors/provisioning/313-helloid-prov-target-nedap-ons-users)
