# research_redaxo_5_17_1

**Introduction:**
During my security research of Redaxo CMS v5.17.1 i found a way how low-level privileged user can obtain admin credensials using stored XSS vulnerability and perform arbitrary code execution on back-end server using administrative account.
**Testing environment:**
- Redaxo CMS version 5.17.1
- AddOns MediaPool v2.14.0, MediaManager v2.16.0, Cronjob v2.11.0 installed
- Admin user h4ckr4v3n and testuser with redacting permisshions.
Test user roles shown in a screenshot:
![fc44f9eac0a8f5f9c3ec9cb3f3a6c2f5.png](:/3719ca5a44ad40318ab1273c313fbca9)

**Killchain:**
Stored XSS.
CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:A/VC:H/VI:N/VA:N/SC:N/SI:N/SA:N 6.8 (Medium)
Let's start with stored XSS. User "testuser" can use MediaPool add-on for creating categories and uploading files. By the way files with dangerous extensions such as php, phar and others are not permitted.

![8104cfc62ee8042307e79dc23d4f499c.png](:/1572d8768b854eb4a7a0e395cab2e1d9)

Nevertheless, there are no restrictions for html document extension. User can upload HTML-page with malicious JavaScript code. Firstly, i've tried to trigger XSS with basic alert() function:

![7e977d21417e30b4907e630cc0e4a6b3.png](:/904db3f87a6d4502b3e6f4f28e0f3e6a)

Message shows us it works. Now we can try to upload HTML-code of the Redaxo login page with malicious JS which handles user input and sends it to my VPS.



![redaxo_xss.png](:/e5abe593752e49e3ab972bcb866fb48c)

After submitting an admin credentials we will receive them on our listening simple Web-server.



![IMG_20240809_163757.png](:/e98a1c8b86434d598b76402c693b05ff)

Using these credentials we can now login to the admin account. It's important to note that these html pages are publicly accessible and can be used for accounts takeover of many users.

Next, I discovered to obtain RCE using admin account.
CVSS:4.0/AV:N/AC:L/AT:N/PR:H/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N 8.6 (High)

First way is PHP Code injection in Templates tab. Admin user can create templates to be used in articles:


![redaxo_template_rce1.png](:/c078000cb3dc4871a0fe9b10eb99613c)

In template editor we need to insert out PHP code to the template field and save it.



![redaxo_template_rce2.png](:/e5cd7201910d49b4ad3fa76ecc819d30)

Now we can add or edit an article via Structure tab and submit our malicious template.



![redaxo_template_rce3.png](:/87f726ea0d8a4e9495a01ae5f01519d0)

And when we will go to the article itself, we will be able to execute arbitrary code via 'cmd' parameter.



![redaxo_template_rce4.png](:/c61302f756e84cd8bbd532c1aa1aed37)

Another way to obtain RCE is using a Cronjob AddOn. Admin user can create cronjobs and inject PHP code inside it.

Firstly, we will install cronjob AddOn. It's important to note that this AddOn is free available and any admin user of any Redaxo instance can install it.

![redaxo_admin_rce.png](:/52efcda6c95845568d4ecbcd53ae3298)

After that we can create new cronjob.


![redaxo_admin_rce1.png](:/b5a1c3dfd6fd4f68bea6d4cb12d80864)

I created code that puts system(); command into PHP file in redaxo working catalog.



![redaxo_admin_rce2.png](:/159a59f4abd249a396821e18c0ff21bf)

After that I can execute arbitrary commands on back-end server using path /redaxo/pwned.php


![redaxo_admin_rce3.png](:/7418cf569cc8435482d37c622410650d)

Note: you can execute cronjobs manually.

**Recommendations:**
You should prevent html pages from being uploaded to the media section, and check them for the use of dangerous JavaScript functions. You should also use security headers such as X-Xss-Protexction, CORS, SOP, CSP, HSTS.
As for executing arbitrary PHP code, you should run cronjob in a sandbox environment and avoid direct interaction with the operating system. This also applies to the templates tab
