---
title: "The governance pain of DevOps + App Registrations"
image: "/assets/img/2025-08-02-the-governance-pain-of-devops+app-registrations/image.png"
categories: [Azure]
tags: ["azure", "app registration"]
---
You have a DevOps team responsible for an web application. The need rights to administrate App Registrations, not just for the app itself, but potentially also for automation agents or pipelines that deploys it. It is hard to merge the DevOps mindset of agility, and the need for governance and control of such important areas as access and identity management.

App Registrations is a identity concern, and for many organisations it represents a clash of cultures. Where DevOps thrives on agility and self-service, identity and access management demands governance, control, and security discipline.

Changes to App Registrations are driven by actions taken by development or infrastructure teams, not by internal IT, who is the one who actually owns and operates Entra ID.

Let's see if we can get back to principles of self service with accountability and ownership by DevOps teams, while stile retaining the security and governance requirements of .

## Core issues with App Registration and DevOps metodology

* __Overly Broad Built-In roles:__ Built-in administrative roles for App Registrations grant fare more privilegies that DevOps teams requires for day to day operations.

* __Disconnecteded Lifecycle:__ The App Registration often outlive the application they represent. This leaves behind orphaned and abandoned registrations that clutter Entra ID and pose potential security risks.

## Operating App Registrations require Entra ID permissions

You need to be carefull before granting teams outside Central IT Operations access to administrate objects in Entra ID. Let's have a look at the different methods for delegating operational rights to App Registrations.

Our requirements are that:

* Privileged administrative tasks, such as managing client secrets can be restrictied,
* Developers or other regular users can't consent to sharing data with the application.
* Adminstrative tasks works with with Privileged Identity Management (PIM).
* Rights to manage App Registrations can be assigned to groups, not just individuals.

From worst to best:

1. __Letting everyone create App Registrations, and manage the App Registrations they have created themselves.__

    By default this is enabled. For most organizations an information security issue. If its still on, turn it off:)

2. __Grant selected users the ability to create, consent and manage all aspects of App registrations.__

    Assigning either [Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-administrator) role or [Cloud Application Administrator](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#cloud-application-administrator) lets them create and manage _all_ aspects off App Registrations tennant wide.

    Anyone with this roles can generate client secrets for any application, and impersonate it. ___Potentially impersonating as applications with permissions higher than the user themselves should have.___ These two roles open a real and possible attack vector. Avoid delegating this role to users outside of the security regiment normally found for highly privileged teams.

3. __Grant selected users the ability to create App Registrations, and manage the App Registrations they have created themselves.__

    Assigning [Application developer](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference#application-developer) gives a user the ability to create app registrations, but limits the user to managing only App Registrations they create (up to 250 to prevent filling Entra ID). Users with this role will be assigned as Owner to the app registration. Ownership is what gives rights to administrate it. This means it has the the same drawbacks as the next alternative, assigning owners.

4. __Assigning Someone as Owner of specific App Registration.__

    Owner rights can only be granted to users _after_ someone with permissions to create App Registration has created it. Issues with this is:
    * Still allows impersonating app registrations by generating client secrets.
    * Only users can be registered as owners. Groups are not supported. Ownership delegations must manually keep up with changes in team staffing.
    * Does not work with PIM.

5. __Creating custom roles with scoped rights following principles of least privilege__

    With this option you can create custom Entra ID role for managing a few selected settings on App registratios. The custom role supports Privileged Identity Management. Roles can be assigned to a specific scope such as tennant wide, or specific app registrations, and be assigned to groups. Let's test it out in the next section.

To summarize:

| Alternative       | Create            | Consent                      | Impersonate                  | Works with PIM                            |
|-------------------|-------------------|------------------------------|------------------------------|-------------------------------------------|
|1. Everyone        | Yes               | App registrations it creates | App registrations it creates | No                                        |
|2. Admin Roles     | Yes               | All app registration         | All app registration         | Yes                                       |
|3. Developer Roles | Yes               | App registrations it creates | App registrations it creates | Only for creating, not for administration |
|4. Assigned owner  | No                | App registrations it owns    | App registrations it owns    | No                                        |
|5. Custom Role     | Can be restricted | Can be restricted            | Can be restricted            | Yes                                       |

### Custom role for management of by developers App Registration

We will be createing custom roles called App Registration Owner, and App Registration Contributor. Permissions for each role:

| Role | Description | Permissions |
|------|-------------|-------------|
| __App Registration Contributor__ | Can read standard properties<br/>Modify audience settings<br/>Update basic properties<br/>Update authentication settings like reply URLs | `microsoft.directory/applications/standard/read`<br/>`microsoft.directory/applications/audience/update`<br/>`microsoft.directory/applications/basic/update`<br/>`microsoft.directory/applications/authentication/update` |
| __App Registration Owner__  | _Same permissions as App Registration Contributor_, pluss:<br/>Update credentials such as Client Key and Client Secret| <br/>`microsoft.directory/applications/credentials/update` |

All available permissions for custom roles can be found in the [Microsoft Entra documentation](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-available-permissions)

#### Custom Role Contributor

![Custom Role Contributor](/assets/img/2025-08-02-the-governance-pain-of-devops+app-registrations/app-registration-contributor.png)

#### Custom Role Owner

![Custom Role Owner](/assets/img/2025-08-02-the-governance-pain-of-devops+app-registrations/app-registration-owner.png)

#### Role assignment

These roles can now be assigned at directory scope (Across all Appregistrations in tenant) or at individual app registrations. It works for both groups and individual users:

![Role Assignment](/assets/img/2025-08-02-the-governance-pain-of-devops+app-registrations/assigning-role.png)

## Infrastructure as Code

Managing the app registration using IaC connects the App Registration to the application lifecycle. It keeps the app registration more visible for the DevOps team, and hopefully makes it less likely to experience orphaned app registrations.

1. __Bicep__

    Can be done with the new MS Graph extension for Bicep. There are a few gotchas. Like onboarding existing app registrations using unique name, or properties you cant change, or should change.

2. __Terraform__

    Possible with the AzAPI provider.

3. __App Registration Operators / Agents__

    This is more of an honorable mention, not suited for everyone. Some organizations deploys agents that looks for deployed applications in cloud infrastructure. These agents detect application changes and can automatically register, updates and removes App Registrations. This might be the option to choose if your organization has a very large pool of applications and development teams.

## Summary

1. Use Managed Identities where you can.
2. Create custom roles with permission to only modify a subset of properties for DevOps teams
3. Leave creation of App Registrations to the Central IT team. They will ensure consistency with naming standards, have been trusted with the privilege of having administrative rights and will grant the DevOps teams groups the rights to administrate it after it has been creates.
4. Concider automation to enforce consistency and reduce manual access and manual labour.
