# HelloID-Conn-Prov-Target-Ysis

> [!IMPORTANT]
> This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.

<p align="center">
  <img src="https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-YsisV2/blob/main/Logo.png?raw=true">
</p>

## Table of contents

- [HelloID-Conn-Prov-Target-Ysis](#helloid-conn-prov-target-Ysis)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Connection settings](#connection-settings)
      - [Correlation configuration](#correlation-configuration)
      - [Field mapping](#field-mapping)
    - [Remarks](#remarks)
      - [`PUT` method for all update actions](#put-method-for-all-update-actions)
      - [Full update within the _update_ lifecycle action](#full-update-within-the-update-lifecycle-action)
      - [Archiving an Ysis-account](#archiving-an-ysis-account)
      - [Conditional Event](#conditional-event)
  - [Getting help](#getting-help)
  - [HelloID Docs](#helloid-docs)

## Introduction

The HelloID-Conn-Prov-Target-Ysis connector creates and updates user accounts within Ysis. The Ysis API is a SCIM based (http://www.simplecloud.info) API and has some limitations for our provisioning process. For more information you can check the Ysis SCIM documentation (https://apihelp.gerimedica.nl/category/scim/).

> [!IMPORTANT]
> It is not possible to change the discipline of an existing account. Therefore, during the `update` life-cycle a change in discipline will launch a conditional event which sends an email to the Ysis administrator.

- In Ysis each account has a discipline that acts as the account type.
- When a person requires a different (or an extra discipline), a new user account must be created with the new discipline. Manual actions by the Ysis administrator are needed.

The following lifecycle actions are available:

| Action             | Description                                                                                                                          |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| create.ps1         | PowerShell _create_ or _correlate_ lifecycle action. If correlated and UpdateOnCorrelate is configured, the update will be processed |
| delete.ps1         | PowerShell _delete_ lifecycle action. Archives the Ysis account, optionally update Username to YsisInitials                          |
| disable.ps1        | PowerShell _disable_ lifecycle action                                                                                                |
| enable.ps1         | PowerShell _enable_ lifecycle action                                                                                                 |
| update.ps1         | PowerShell _update_ lifecycle action. Conditional event on discipline change.                                                        |
| subPermissions.ps1 | PowerShell _subPermission_  lifecycle action. Add Ysis module based on mapping.csv                                                   |
| configuration.json | Default _configuration.json_                                                                                                         |
| fieldMapping.json  | Default _fieldMapping.json_                                                                                                          |

## Getting Started

### Prerequisites

- [ ] The outgoing IP address of the HelloID agent server must be whitelisted by GeriMedica.
- [ ] Mapping between function and discipline.

> [!TIP]
> You can validate the outgoing IP address on the HelloID agent server with the following PowerShell script:
> ```powershell
> $ip = Invoke-RestMethod -uri "https://ipinfo.io/json" -method get
> Write-Verbose -Verbose "$($ip.ip)"
> ```

### Connection settings

The following settings are required to connect to the API.

| Setting                 | Description                                                                   |
| ----------------------- | ----------------------------------------------------------------------------- |
| ClientID                | The ClientId to connect to the Ysis API                                       |
| ClientSecret            | The ClientSecret to connect to the Ysis API                                   |
| BaseUrl                 | The URL to the Ysis environment. Example: https://company.acceptatie2.ysis.nl |
| DefaultModule           | The default module code. Default value: `YSIS_CORE`                           |
| MappingFile             | The mapping between function and discipline                                   |
| UpdatePersonOnCorrelate | This will update the account in the target application during correlation     |
| UpdateUsernameOnDelete  | Update username to the YsisIntials when archiving Ysis account                |
| IsDebug                 | When toggled, debug logging will be displayed                                 |

#### Correlation configuration

The correlation configuration is used to specify which properties will be used to match an existing account within _HelloID-Conn-Prov-Target-Ysis to a person in _HelloID_.

To properly setup the correlation:

1. Open the `Correlation` tab.

2. Specify the following configuration:

    | Setting                   | Value            |
    | ------------------------- | ---------------- |
    | Enable correlation        | `True`           |
    | Person correlation field  | ``               |
    | Account correlation field | `EmployeeNumber` |

> [!TIP]
> _For more information on correlation, please refer to our correlation [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems/correlation.html) pages_.

#### Field mapping

The field mapping can be imported by using the [_fieldMapping.json_](./fieldMapping.json) file.

### Remarks

#### `PUT` method for all update actions

All update actions use an `HTTP.PUT` method. This means that the full account object will be send to Ysis. For both the _enable_ and _disable_ lifecycle actions, we first retrieve the account, update the `active` property accordingly and send back the full object.

#### Full update within the _update_ lifecycle action

The _update_ lifecycle action now supports a full account update. Albeit, the update itself is a `PUT`. This means that the __full__ object will be updated within Ysis. Since the update process is also supported from the _create_ lifecycle action, this might have unexpected implications.

Some values may not be available in HelloID because they are not available in the HR system. If these values are added manually in Ysis you need to make sure HelloID sends back the current value in the update.ps1 script. Example:

```powershell
# Set AGB to existing if null or empty
if (!([string]::IsNullOrEmpty($previousAccount.AgbCode)) -and [string]::IsNullOrEmpty($account.AgbCode)) {
    $account.AgbCode = $previousAccount.AgbCode
}
```

#### Archiving an Ysis-account

HelloID can archive a Ysis account, but can't dearchive an Ysis account.  HelloID will update the Ysis username to the YsisIntials if `updateUsernameOnDelete` is `enabled` i to make sure a new account can be created. If updating the username is not used. Then this can result in messages regarding existing usernames. The archived account then needs to be dearchived manually or corrected by setting a dummy username.

#### Conditional Event
A conditional event needs to be set up based on changes of the discipline. On this event a notification can be configured to send an e-mail to the Ysis-administrator.

> [!TIP]
> How to configure:
> 1. Make sure `Discipline` is added in the field mapping and the option `Use in notifications` is on.
> 2. Go to Business Custom events, create a new custom event. Select the Ysis connector, action `Account update` and add a condition with field `Discipline` is updated.
> 3. Go to Notifications Configuration, create a new notification. Select your Ysis custom event. Import the [_conditional-notification.mjml_](./conditional-notification.mjml) template.
>
> _For more information custom events, please refer to our [documentation](https://docs.helloid.com/en/provisioning/notifications--provisioning-/custom-notification-events--conditional-notifications-.html) pages_.

## Getting help

> _For more information on how to configure a HelloID PowerShell connector, please refer to our [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems.html) pages_

> _If you need help, feel free to ask questions on our [forum](https://forum.helloid.com)_

## HelloID Docs

The official HelloID documentation can be found at: https://docs.helloid.com/