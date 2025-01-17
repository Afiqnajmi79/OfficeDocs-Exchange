---
localization_priority: Normal
description:
ms.topic: conceptual
author: mattpennathe3rd
ms.author: v-mapenn
ms.assetid:
ms.date:
ms.reviewer:
title: Enable or disable SMTP AUTH
ms.collection:
- exchange-online
- M365-email-calendar
audience: ITPro
ms.service: exchange-online
manager: serdars

---

# Enable or disable authenticated client SMTP submission (SMTP AUTH) in Exchange Online

Client SMTP email submissions (also known as _authenticated SMTP submissions_) are used in the following scenarios in Office 365:

- POP3 and IMAP4 clients. These protocols only allow clients to _receive_ email messages, so they need to use authenticated SMTP to _send_ email messages.

- Applications, reporting servers, and multifunction devices that generate and send email messages.

The SMTP AUTH protocol is used for client SMTP email submission (typically on TCP port 587). SMTP AUTH doesn't support modern authentication (Modern Auth), and only uses basic authentication, so all you need to send email messages is a username and password. This makes SMTP AUTH a popular choice for attackers to send spam or phishing messages using compromised credentials.

Virtually all modern email clients that connect to Exchange Online mailboxes in Office 365 (for example, Outlook, Outlook on the web, iOS Mail, Outlook for iOS and Android, etc.) don't use SMTP AUTH to send email messages.

Therefore, we highly recommend that you disable SMTP AUTH in your Exchange Online organization, and enable it only for the accounts (that is, mailboxes) that still require it. There are two settings that can help you do this:

- An organization-wide setting to disable (or enable) SMTP AUTH.

- A per-mailbox setting that overrides the tenant-wide setting.

Note these settings only apply to mailboxes that are hosted in Exchange Online (Office 365).

## Disable SMTP AUTH in your organization

You can only disable (or enable) SMTP AUTH globally for your organization by using [Exchange Online PowerShell](https://go.microsoft.com/fwlink/p/?LinkId=396554).

To disable SMTP AUTH globally in your organization, run the following command:

```
Set-TransportConfig -SmtpClientAuthenticationDisabled $true
```

**Note**: To enable SMTP AUTH if it's already disabled, use the value `$false`.

### How do you know this worked?

To verify that you've globally disabled SMTP AUTH in your organization, run the following command and verify the value of the **SmtpClientAuthenticationDisabled** property is `True`:

```
Get-TransportConfig | Format-List SmtpClientAuthenticationDisabled
```

## Enable SMTP AUTH for specific mailboxes

The per-mailbox setting to enable (or disable) SMTP AUTH is available in the Microsoft 365 admin center or [Exchange Online PowerShell](https://go.microsoft.com/fwlink/p/?LinkId=396554).

### Use the Microsoft 365 admin center to enable or disable SMTP AUTH on specific mailboxes

1. Open the [Microsoft 365 admin center](https://admin.microsoft.com) and go to **Users** \> **Active users**.

2. Select the user, and in the flyout that appears, click **Mail**.

3. In the **Email apps** section, click **Manage email apps**.

4. Verify the **Authenticated SMTP** setting: unchecked = disabled, checked = enabled.

   When you're finished, click **Save changes**.

### Use Exchange Online PowerShell to enable or disable SMTP AUTH on specific mailboxes

Use the following syntax:

```
Set-CASMailbox -Identity <MailboxIdentity> -SmtpClientAuthenticationDisabled <$true | $false | $null>
```

The value `$null` indicates the setting for the mailbox is controlled by the global setting on the organization. You use the values `$true` (disabled) or `$false` (enabled) to _override_ the organization setting. The mailbox setting takes precedence over the organization setting.

This example enables SMTP AUTH for mailbox sean@contoso.com.

```
Set-CASMailbox -Identity sean@contoso.com -SmtpClientAuthenticationDisabled $false
```

This example disables SMTP AUTH for mailbox chris@contoso.com.

```
Set-CASMailbox -Identity chris@contoso.com -SmtpClientAuthenticationDisabled $true
```

### Use Exchange Online PowerShell to enable or disable SMTP AUTH on multiple mailboxes

Use a text file to identify the mailboxes. Values that don't contain spaces (for example, alias, email address, or account name) work best. The text file must contain one mailbox on each line like this:

> akol@contoso.com <br> tjohnston@contoso.com <br> kakers@contoso.com

The syntax uses the following two commands (one to identify the mailboxes, and the other to enable SMTP AUTH for those mailboxes):

```
$<VariableName> = Get-Content "<text file>"
$<VariableName> | foreach {Set-CASMailbox -Identity $_ -SmtpClientAuthenticationDisabled <$true | $false | $null>}
```

This example enables SMTP AUTH for the mailboxes specified in the file C:\My Documents\Allow SMTP AUTH.txt.

```
$Allow = Get-Content "C:\My Documents\Allow SMTP AUTH.txt"
$Allow | foreach {Set-CASMailbox -Identity $_ -SmtpClientAuthenticationDisabled $false}
```

**Note**: To disable SMTP AUTH for the mailboxes, use the value `$true`. To return control to the organization setting, use the value `$null`.

### How do you know this worked?

To verify that you've enabled or disabled SMTP AUTH for a specific mailbox, do any of the following steps:

- **Individual mailboxes in the Microsoft 365 admin center**: Go to **Users** \> **Active users** \> select the user \> click **Mail** \> click **Manage email apps** and verify the value of **Authenticated SMTP** (checked = enabled, unchecked = disabled).

- **Individual mailboxes in Exchange Online PowerShell**: Replace `<MailboxIdentity>`, with the name, alias, email address or account name of the mailbox, run the following command, and verify the value of the **SmtpClientAuthenticationDisabled** property (`False` = enabled, `True` = disabled, blank = use organization setting).

  ```
  Get-CASMailbox -Identity <MailboxIdentity>  | Format-List SmtpClientAuthenticationDisabled
  ```

- **All mailboxes where SMTP AUTH is disabled**: Run the following command:

  ```
  $Users = Get-CASMailbox -ResultSize unlimited
  $Users | where {$_.SmtpClientAuthenticationDisabled -eq $true}
  ```

- **All mailboxes where SMTP AUTH is enabled**: Run the following command:

  ```
  $Users = Get-CASMailbox -ResultSize unlimited
  $Users | where {$_.SmtpClientAuthenticationDisabled -eq $false}
  ```

- **All mailboxes where SMTP AUTH is controlled by the organization setting**: Run the following command:

  ```
  $Users = Get-CASMailbox -ResultSize unlimited
  $Users | where {$_.SmtpClientAuthenticationDisabled -eq $null}
  ```
