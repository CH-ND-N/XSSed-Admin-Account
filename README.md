# XSSed-Admin-Account
# How I XSSed Admin Account
Hello Guys, recently I encountered with some Stored XSS on a web application which helped me takeover any account on the application.
Although some of the Stored XSS was easy to find, the impact which it had on the application was near critical and hence I’ll pick one of the cases in this post to explain the Impact.

# Case Study: Stored XSS to Account Takeover
The application ******* was under test was a three-tier web application – Presentation tier (Front-End/User Interface), Application Tier (Functional Logic) and Data-Tier (Databases). The application is a CRM application used for scheduling meetings, phone calls and sending Emails. The application used tokens to authenticate users when the user logged in the application. Using tokens added a layer of protection from CSRF and hence prevented requests to be sent on behalf of the user.

The token was generated as follows:
The username and some random string were encoded into base64 as-

admin:d4sb6bdfhfyr7hasdr434df was encoded as YWRtaW46ZDRzYjZiZGZoZnlyN2hhc2RyNDM0ZGY=

This token was added to every request and removing the token would send the request as unauthorized.
On checking the request to the site:

GET /******/ HTTP 1.1
Host: localhost
Authorization: Basic YWRtaW46ZDRzYjZiZGZoZnlyN2hhc2RyNDM0ZGY=
Espo-Authorization: YWRtaW46ZDRzYjZiZGZoZnlyN2hhc2RyNDM0ZGY=
Cookie: auth-username=admin; auth-token= d4sb6bdfhfyr7hasdr434df

We can see that the two major elements to form the Auth token were inside the cookie of the request. The Authorization token was constructed as follows:

Base64 encoding of (auth-username:authtoken)

Now, all we needed was to get a Reflected or Stored XSS on the application to take over any user as the cookies which form the token are not set as HttpOnly and can be captured by an attacker if he finds a way to deliver the malicious JavaScript.

The application had a preference tab where the user could set his email signature which is added to every email. The Email signature had various markdown methods. One of the features in the markdown which got my attention was the Code View feature on which you could try inserting HTML code inside the signature for better looks.

# The first attempt:
<svg/onload=alert(1)> was inserted in the code view field. On clicking the close code view feature, the alert box popped. This worked at that moment but on clicking on save, the event handlers were stripped in the application

Payload: <svg/onload=alert(1)>

Final response: <svg/[data-handler-stripped]=alert(1)>

# The second attempt:
XSS via Html Injection and Iframe. <a> tags and <iframe> tags were accepted by the application. Using the payload <a href=javascript:alert(1)>test</a> or the same with Iframe, the tags were inserted into the application but stripped the href or src values.

Payload: <a href=javascript:alert(1)>test</a>

Response:<a>test</a>

Payload: <iframe src=javscript:alert(1)></iframe>

Response: <iframe></iframe>

After this all payloads from Payloads all the things were tried, but there was no luck. While scrolling through the section about XSS in the same GitHub page, I stumbled across this polyglot XSS payload from Somdev Sangwan.

–>'”/></sCript><svG x=”>” onload=(co\u006efirm)“>

As this payload was inserted, it was seen that the XSS was permanently stored on the Preference page.
The code was then modified as such to steal cookies of the user.

–> “/><svg x=”>” onload=”(co\u006efirm)(document.cookie)”></svg>

The payload was very impactful to generate the stored XSS inside the Email as it would fire when the victim would reply or forward the Email having XSS inside the Email Signature.

Now all the attacker needed to do was send an Email to the admin of the application who when replied to the mail executed the malicious JavaScript. The attacker would now steal the cookies of admin and construct the Authorization token of the admin and hence completely take over his account.


# BugBounty Tips:
Xss using css:
<style>img{background-image:url(‘javascript:alert(1)’)}</style>


firewall bypass:

<style>*{background-image:url(‘\6A\61\76\61\73\63\72\69\70\74\3A\61\6C\65\72\74\28\6C\6F\63\61\74\69\6F\6E\29’)}</style>
