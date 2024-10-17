# Integrate MDaemon Webmail Sign-In with Intranet Site

There are times that you want to integrate MDaemon Webmail Sign-In with your intranet site. This is a simple example of how to do that.

## Server Setup for Intranet Site
If you wish to provide access to the Webmail server from an intranet site without going to the MDaemon Webmail page, you could do so by adding features to your intranet site that use the WorldClientAPI.

In the file `MDaemon\WorldClient\Domains.ini` add the following setting line to the [Default:Settings] section:

`WorldClientAPI:AllowTheseExternalOrigins=%HOSTS%`

Where `%HOSTS%` is a semi-colon separated list of hosts to allow API access from.

Example: https://mail.mywebmailsite.com;https://mail.mywebmailsite2.com

Without this setting, only the web address being hosted by MDaemon Webmail will be able to access the API.

The WorldClient API Documentation is located at `MDaemon\Docs\API\WorldClient\WorldClientAPI.html`

## Client Setup
Using your preferred HTTP client, you can make a request to the WorldClientAPI to authenticate a user.

If [Session] UseCSRFToken=Yes in the `MDaemon\WorldClient\WorldClient.ini` file, then you need to make two requests to the server.

The first request will create a login cookie to be sent with the authentication request.

The second request will be the authentication request.

For example, in JavaScript do the following:
```javascript
  const some_uuid = "data string that is a unique identifier";

  fetch(`https://mail.mywebmailsite.com/WorldClientAPI/settings/logon_settings=login_token&data=${some_uuid}`, {
    method: "GET"
  });
```
  `some_uuid` needs to be a unique string that's saved for the next request.

  Make sure to save the cookies from the first request and send them with the second request if you're not using the browser.

```javascript
  fetch("https://mail.mywebmailsite.com/WorldClientAPI/authenticate/basic/", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      user: "user@mywebmailsite.com",
      password: "password",
      data: some_uuid,
      terms_of_use_acknowledged: true, // if required
    })
  }).then(res => res.json()).then(response => {

  });

```

Possible `response` values:

#### User is authenticated, set the session and redirect to the webmail page /webmail/mail
```
  {
    "authed": true,
    "session": @string {session_id},
    "csrf_token": @string, // base64 encoded token for POST, PUT, DELETE requests
    "user": @string, // the users actual email address
    "remembered": @boolean
  }
```
#### User is almost authenticated, redirect to the Two Factor Auth page /webmail/2fa
```
  {
    "authed": false,
    "warning": "TwoFactorAuthRequired",
    "token": @string,
    "user": @string, // the users actual email address
    "partial": @string // a partially obfuscated email address where the code is sent if the user has this configured
  }
```
#### User is authenticted, redirect to the Two Factor Auth Setup page /webmail/2fasetup
```
  {
    "authed": true,
    "session": @string,
    "csrf_token": @string, // base64 encoded token for POST, PUT, DELETE requests
    "warning": "TwoFactorAuthSetupRequired",
    "token": @string,
    "user": @string // the users actual email address
  }
```
#### User is not authenticated, the user needs to change the password before sign-in
```
  {
    "authed": false,
    "warning": "OldPassword",
    "token": @string
  }
```

## Working Example

Drop the `intranet.html` file from this repository into your MDaemon\WorldClient\HTML directory and use it as a starting point.
It's located at [https://github.com/maemon-technologies/intranet-integration/blob/main/intranet.html](intranet.html)

To make use of the page, pass the `user` and `password` parameters to it from your intranet site like this:

```html

  <script>
    function webmailSignIn() {
      const signInLink = document.getElementById("signInLink");
      // using whatever method you need in order to get the user and password
      signInLink.href = "https://mail.mywebmailsite.com/intranet.html?user=user@mywebmailsite.com&password=password";
      signInLink.click();
    }


  </script>

  <button onclick="webmailSignIn()">Sign In</button>
  <a id="signInLink" target="_blank"></a>

```
The `user` and `password` parameters will not remain in the URL for long, but they will be visible, so base64 encoding them or using encryption is recommended.

Change the intranet.html file to decode or decrypt them before posting them to the WorldClientAPI server. 

```javascript
  // your file
  const user = btoa("user@mywebmailsite.com");
  const password = btoa("password");


  // in intranet.html
  window.onload = function () {
    let user = "", password = "";
    if (document.location.search.includes("user")) {
      user = document.location.search.split("user=")[1].split("&")[0];
      user = atob(user);
      document.getElementById("user").value = user;
    }
    if (document.location.search.includes("password")) {
      password = document.location.search.split("password=")[1].split("&")[0];
      password = atob(password);
      document.getElementById("password").value = password;
    }
    if (user && password) {
      SignIn(user, password);
    }
  };
```

An additional security measure that you could take is to use the hash portion of the URL to pass the `user` and `password` to `intranet.html` instead of the query portion, since the hash portion is not visible to the server. The file would need to be modified to get the `user` and `password` from the hash portion of the URL.

For example: 

`https://mail.mywebmailsite.com/intranet.html#user=user@mywebmailsite.com&password=password`

```javascript
  window.onload = function () {
    let user = "", password = "";
    if (document.location.hash.includes("user")) {
      user = document.location.hash.split("user=")[1].split("&")[0];
      document..getElementById("user").value = user;
    }
    if (document.location.hash.includes("password")) {
      password = document.location.hash.split("password=")[1].split("&")[0];
      document.getElementById("password").value = password;
    }
    document.location.hash = "";
    if (user && password) {
      SignIn(user, password);
    }
  };
```

The sample `intranet.html` file does not have styling or handling of Two Factor Authentication, Change Password, or other features.
