# Integrate MDaemon Webmail Sign-In with Intranet Site

There are times that you want to integrate MDaemon Webmail Sign-In with your intranet site. This is a simple example of how to do that.

## Server Setup
In the file `MDaemon\WorldClient\Domains.ini` add the following setting line to the [Default:Settings] section:

`WorldClientAPI:AllowTheseExternalOrigins=%HOSTS%`

Where `%HOSTS%` is a semi-colon separated list of hosts to allow API access from.

Example: https://www.example.com;https://www.example2.com

Without this setting, only the web address being hosted by MDaemon Webmail will be able to access the API.

If the host address for the intranet is the same as the host address for MDaemon Webmail, you do not need to include the above setting.

## Client Setup
Using your preferred HTTP client, you can make a request to the WorldClientAPI to authenticate a user.


If [Session] UseCSRFToken=Yes in the MDaemon\WorldClient\WorldClient.ini file, then you need to make two requests to the server.

The first request will get you a login token to be sent with the authentication request.

The second request will be the authentication request.

For example, in JavaScript do the following:
```javascript
  const some_uuid = "data string that is a unique identifier";
  fetch(`https://www.example.com/WorldClientAPI/settings/logon_settings=login_token&data=${some_uuid}`, {
    method: "GET"
  });
```
  `some_uuid` needs to be a unique string that's saved for the next request.

  Make sure to save the cookie from the first request and send it with the second request.

```javascript
  fetch("https://www.example.com/WorldClientAPI/authenticate/basic/", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      user: "user@example.com",
      password: "password",
      data: some_uuid,
      terms_of_use_acknowledged: true, // if required
    })
  }).then(res => res.json()).then(response => {

  });

```

Possible responses:
```json
  User is authenticated, set the session cookie and redirect to the webmail page /webmail/mail
  {
    "authed": true,
    "session": @string {session_id},
    "csrf_token": @string, // base64 encoded token for POST, PUT, DELETE requests
    "user": @string {user}, // the users actual email address
    "remembered": @boolean
  }

  User is almost authenticated, redirect to the Two Factor Auth page /webmail/2fa
  {
    "authed": false,
    "warning": "TwoFactorAuthRequired",
    "token": @string,
    "user": @string, // the users actual email address
    "partial": @string // a partially obfuscated email address where the code is sent if the user has this configured
  }

  User is authenticted, redirect to the Two Factor Auth Setup page /webmail/2fasetup
  {
    "authed": true,
    "session": @string,
    "csrf_token": @string, // base64 encoded token for POST, PUT, DELETE requests
    "warning": "TwoFactorAuthSetupRequired",
    "token": @string,
    "user": @string // the users actual email address
  }

  User is not authenticated, they need to change their password before sign-in
  {
    "authed": false,
    "warning": "OldPassword",
    "token": @string
  }
```

## Working Example

You could also drop the `intranet.html` file into your MDaemon\WorldClient\HTML\webmail directory and use it as a starting point.

To make use of the file, pass the `user` and `password` parameters to the page like this:

```html
  document.location.href = "https://www.example.com/webmail/intranet.html?user=user@example.com&password=password";

  
```
