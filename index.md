## Welcome to Tata Digital

This guide is a quick start to integreta the brand with TCP across systems.

You can integrate with below Apps

- Single Sign-on (SSO)
- Golden Recrod (GR)
- Loylaty
- Offer Engine
- Online Payment Experiance Layer (OPEL)
- Tata Pay (TPM)

### Single Sign-on (SSO)

# Integrating Tata Digital Sign-In to Web App

Tata Digital Sign-In manages the OAuth 2.0 flow and token lifecycle, simplifying
your integration with Tata Digital APIs. This document describes how to complete a
basic Tata Digital Sign-In integration.

## 1. 1 Before you begin

Before you can integrate Tata Digital Sign-In into your website, you must create a
client ID, which you need to call the sign-in API. Most of the client IDs for brands
are already created and can contact SSO team to get that for your brand and
platform.

To create a Tata Digital API Admin Console project and client ID, with Tata Digital
Team. When you configure the project, select the Web / Android / iOS client type
and specify the origin URI of your app.

After configuration is complete, take note of the client ID that was created. You will
need the client ID to complete the next steps. (A client secret is also created, but
you need it only for server-side operations.)

## 1.2 Load the Tata Digital Platform Library

You must include the Tata Digital Platform Library on your web pages that integrate
Tata Digital Sign-In.

<script src="https://dev-account.tatadigital.com/tdl-sso.js" async
defer></script>

## 1.3 Specify app's client ID

Specify the client ID you created for your app in the Tata Digital Admin with
the tdl-sso-client_id meta element in head of the html page.

<meta name="tdl-sso-client_id" content="YOUR-CLIENT-ID" />


## 1.4 Add a Tata Digital Sign-In button:

The easiest way to add a Tata Digital Sign-In button to your site is to use an
automatically rendered sign-in button. With only a few lines of code, you can add a
button that automatically configures itself to have the appropriate text, logo, and
colors for the sign-in state of the user and the scopes you request. To create a Tata
Digital Sign-In button that uses the default settings, add div element with the
class tata-signin2 to sign-in page:

<div class="tata-signin2" data-onsuccess="onSignIn"></div>

## 2.1 When does not session exists:

On clicking this button user will be redirected to:

https://dev-account.tatadigital.com/?clientId=<CLIENT-
ID>&redirectURL=<current_window_location>

Custom button can also be created for this and you can do redirection manually
whenever you have a requirement for login. In this case you and give any callback
URL of your choice, but the base domain is the same. If the redirectURL contains
query paraments or anchors then it should be passed as
encodeURIComponent(yourRedirectUrl) and clientId same as above.

Here login or enrolment journey will take place where client login methods will be
supported. After successful login the user be redirected back to brand website with
authCode and codeVerifier in query parameters as follows:

https://www.brand-website.com/?authCode=<authCode>
&existingUser=1&codeVerifier=<codeVerifier>

The authCode and and codeVerifier should be used immediately to get
accessToken, refreshToken and idToken as explained in the following sections.

Note: If you are testing on local please use https://localhost: 3000 as your client
domain as it is CORS enabled in SSO API management.

The session created after successful login is at the domain https://dev-
account.tatadigital.com. If you want session cookie brand website domain you may
create and manage that.


## 2.2 Get Auth Tokens: When session exists (Auto-login)

After you have signed in a user with Tata Digital using the default scopes, you can
access the user's Tata Digital ID, name, profile URL, and email address, phone no.

To retrieve profile information for a user, use the getTdlSsoToken() method.

function getTdlSsoToken ({ authCode, codeVerifier }) {
console.log(authCode, codeVerifier);
}

Then, send the token to SSO server with an HTTPS POST request to get idToken:

async function getTdlSsoToken({ authCode, codeVerifier }) {
console.log('Logging tdl tokens', authCode, codeVerifier);
if(!authCode) return;
fetch('https://dapi.tatadigital.com/api/v1/sso/access-
token/'+authCode, {
mode: 'cors',
method: 'POST',
body: JSON.stringify({ codeVerifier }),
headers: {
'Content-Type': 'application/json',
'Access-Control-Allow-Origin': '*',
client_id: "TCP-WEB-APP",
client_secret: "c2632ea1-27be-44ac-a6c8-5f6335048003"
}
}).then(async res => console.log(await res.json()));
}

The sample response is as follows:
{
"success": "Token Generated",
"accessToken": "c95bd15b-ddc5-4c58- 9171 - f0b934ba74b7",
"refreshToken": "6cfb8bbc-a2f5- 4125 - 8331 - a437aff395f5",
"idToken": {
"customerHash": "48e269acc7288bf340bbdd9ad46d231e",
"firstName": "Abhishek",
"lastName": "Kanthed",
"email": "abhishek@kanthed.com",
"phone": 8828291901
}
}


## 3. 1 Sign out a user

You can enable users to sign out of your app without signing out of Tata Digital by
adding a sign-out button or link to your site. To create a sign-out link, attach a

function that calls the tdlSsoAuth.signOut() method to the link's onclick event.

<a href="#" onclick="signOut();">Sign out</a>
<script>
function signOut() {
const accessToken = localStorage.getItem("access_token");
tdlSsoAuth.signOut(accessToken).then(() => console.log('User signed
out.')); }
</script>

## 4.1 Authenticate with a backend server

If you use Tata Digital Sign-In with an app or site that communicates with a backend
server, you might need to identify the currently signed-in user on the server. To do
so securely, after a user successfully signs in, send the user's access token to your
server using HTTPS. Then, on the server, verify the integrity of the access token and
use the user information contained in the token to establish a session or create a
new account.

### Send the access token to Tata Digital server as Bearer Token:

After a user successfully signs in, get the user's access token.

Then, send the access token to the server with all subsequent requests for user
validation:

var xhr = new XMLHttpRequest();
xhr.open('POST', 'https://dapi.tatadigital.com/api/v1/example');
xhr.setRequestHeader('Authorization', 'Bearer <access_token>');
xhr.onload = function() {
console.log('Success Response: ' + xhr.responseText);
};
xhr.send();


## 4 .2 Verify the integrity of the access token:

After you receive the access token by HTTPS POST, you must verify the integrity of
the token. To verify that the token is valid, ensure that you make the following API
call:

Headers:
client_id: ‘CLIENT-ID’

Request: GET
https://dapi.tatadigital.com/api/v1/sso/validate-token/{access_token}

If the token is properly signed, you will get a HTTP 200 response, where the body
contains the JSON-formatted token claims. Here's an example response:

{
"success": "Valid access token",
"customerHash": "5f35fd67c673699b98ec50327ebde2ed",
"ttl": "86373"
}
// Here ttl is the expiry time of access token in seconds.

## 4.3 Getting new access token when old one expires:

Access token is short lived, but for longer uninterrupted session for user you can get
a new access token by making the following API call:

Headers:
client_id: ‘CLIENT-ID’
client_secret: ‘CLIENT-SECRET’

Request: POST
https://dapi.tatadigital.com/api/v1/sso/refresh-token

Body: { refreshToken: “<refresh_token>” }

If the refresh token is properly signed, you will get a HTTP 200 response, where the
body contains the JSON-formatted new access token. Here's an example response:

{
"access_token": "e3975607-2ccb-45f6-bf69-ef6853a086d1",
"token_type": "bearer",
"expires_in": 86399
}
// Here ttl is the expiry time of refresh token in seconds.






### Golden Recrod (GR)


### Loylaty


### Offer Engine


### Online Payment Experiance Layer (OPEL)


### MSD





```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/vktdl/tcpdocs/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
