# the requests module


your lovely host is here to explain to you what requests module does


- in a sense when you're connecting to the internet, you use protocols like HTTP to fetch webpages, fetch json data.
- now the core concept is that requests modlue CAN be written in sockets, infact it is written in sockets, but the point is that it acts as a very cool abstraction layer of sockets to hide away the advanced stuff.

heres some simple things you can do, interact with REST apis and all


# Sending form data
form_data = {"username": "alice", "login": "success"}
response = requests.post("https://httpbin.org", data=form_data)

# Sending JSON data to a REST API
json_data = {"title": "Buy groceries", "completed": False}
response = requests.post("https://typicode.com", json=json_data)

also theres a bunch of methods like OPTIONS etc. which honestly wont be needing unless you're really struggling in life.


when you normally process a response object you need 3 things almost always

```py

request.text # whats the text returned (useful for debugging)

request.content # actual content

request.status_code # did it even return stuff properly

request.json() # i want jsonified content

response.ok # is the request ok enough to come back

```


another thing thats very cool about the requests module is i dont normally touch the requests module myself

i give a curl bash representation of the raw request and curlconverter.com converts it its pretty neat


along with that it converts some important details which you have to keep 

### headers 

```py
# headers

custom_headers = {
    "User-Agent": "MyCustomApp/1.0",
    "Authorization": "Bearer YOUR_SECRET_TOKEN"
}
response = requests.get("https://example.com", headers=custom_headers)
```

this is basically saying "hey im this machine" and is required for authentication authorization whatever, basically don't be malicious hacker trying to do CSRF


### sessions 

heres why you need sessions in your life


```py
# Using a session to persist a login state
with requests.Session() as session:
    session.auth = ('user', 'pass')
    session.headers.update({'x-test': 'true'})
    
    # Both requests will carry the auth and custom headers automatically
    response1 = session.get('https://httpbin.org')
    response2 = session.get('https://httpbin.org')
```

in general every request has its own blank slate but a session holds ALL of its cookies and data given back by the host website. so if you're trying to web crawl (like i have) over sites that usually require one concurrent session with configs that require matching details always, then this is the key

### error handling

```py

try:
    # Timeout if the server doesn't respond within 3.5 seconds
    response = requests.get("https://example.com", timeout=3.5)
    
    # Raises an HTTPError if the response was a 4xx or 5xx error
    response.raise_for_status() 
except requests.exceptions.Timeout:
    print("The request timed out.")
except requests.exceptions.HTTPError as err:
    print(f"HTTP error occurred: {err}")
```

self explanatory.

