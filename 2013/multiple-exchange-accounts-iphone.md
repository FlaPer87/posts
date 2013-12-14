<!---
$"metadata"$
{"md": true, "upload_date": "2010-01-01 14:15:21", "title": "Multiple Exchange accounts on IPhone", "draft": false, "slug": "multiple-exchange-accounts-iphone", "tags": ["iphone", "contacts", "exchange"]}
$"metadata"$
-->
I did it like this:

(You'll need ssh on the IPhone)

1-) Configure one of the exchange accounts on the iphone

2-) SSH you're IPhone and cd the /private/var/mobile/Library/Preferences folder

3-) Execute "plutil -convert xml1 com.apple.accountsettings.plist" (This will convert the plist file to xml)

4-) Copy the converted file to your local machine

5-) Remove the configured Exchange account from the IPhone and configure the new one.

6-) Repeat steps 3, 4 and 5 for each Exchange account.

7-) Merge all the files you copied locally.

8-) Copy the new file (The merged one) to the IPhone (Same folder and same name)

9-) Go to the IPhone mail settings and try disabling and enabling one of the accounts (This will refresh the mail/contacts/calendar settings)

10-) Enjoy