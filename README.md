# xander (work in progress)
An easy to use web application development library and framework.

## Installation
```nimble install https://github.com/sunjohanday/xander.git```

[OPTIONAL] If you want to install Xander CLI
```~/.nimble/pkgs/xander-[VERSION]/xander/install.sh```

A basic example:
```nim
import xander

get "/":
  respond "Hello World!"

runForever(3000)
```
More examples can be found in the ```examples``` folder.

## The Gist of It
xander injects variables for the developer to use in request handlers. These variables are:

- request, the http request
- data, contains data sent from the client such as get parameters and form data (shorthad for JsonNode)
- headers, for setting response headers (see request.headers for request headers)
- cookies, for accessing request cookies and setting response cookies
- session, client specific session variables
- files, uploaded files

```nim
# Request Handler definition
type
  RequestHandler* =
    proc(request: Request, data: var Data, headers: var HttpHeaders, cookies: var Cookies, session: var Session, files: var UploadFiles): Response {.gcsafe.}
```

These variables do a lot of the legwork required in an effective web application.

## Serving files
To serve files from a directory (and its sub-directories)
```nim
# app.nim
# the app dir contains a directory called 'public'
serveFiles "/public"
```
```html
<img src="/public/image.jpg" height="300"/>
<script src="/public/js/main.js"></script>
<link rel="stylesheet" href="/public/css/main.css"/>
```

## Templates
xander provides support for templates, although it is very much a work in progress.
To serve a template file:
```nim
# Serve the index page
respond tmplt("index")
```
The default directory for templates is ```templates```, but it can also be changed by calling
```nim
setTemplateDirectory("views")
```
By having a ```layout.html``` template one can define a base layout for their pages.
```html
<!DOCTYPE html>
<html>
  <head>
    <title>{[title]}</title>
  </head>
  <body>
    {[ content ]}
    {[ template footer ]}
  </body>
</html>
```
In the example above, ```{[title]}``` is a user defined variable, whereas ```{[ content ]}``` is a xander defined variable, that contains the contents of a template file. To include your own templates, use the ```template``` keyword ```{[template my-template]}```. You can also include templates that themselves include other templates.

```html
<!-- templates/footer.html -->
<footer>
  
  <a href="/">Home</a>

  <!-- templates/contact-details.html -->
  {[ template contact-details ]}

</footer>
```
You can also seperate templates into directories. The closest layout file will be used; if none is found in the same directory, parent directories will be searched.
```
appDir/
  app.nim
  ...
  templates/
    
    index.html    # Root page index
    layout.html   # Root page layout

    register/
      index.html  # Register page index
                  # Root page layout 

    admin/
      index.html  # Admin page index
      layout.html # Admin page layout

    normie/
      index.html  # Client page index
      layout.html # Client page layout
    
```

### For loops
For loops are supported in Xander templates. This is still very much a work in progress.
```html
<body>

  <table>
    <tr>
      <th>Name</th>
      <th>Age</th>
      <th>Hobbies</th>
    </tr>

    {[ for person in people ]}
    <tr>
      <td>{[ person.name ]}</td>
      <td>{[ person.age ]}</td>
      <td>
        <ul>
          {[ for hobby in person.hobbies ]}
          <li>{[ hobby ]}</li>
          {[ end ]}
        </ul>
      </td>
    </tr>
    {[ end ]}

  </table>

</body>
```

### Template variables
xander provides a custom type ```Data```, which is shorthand for ```JsonNode```, and it also adds some functions to make life easier. To initialize it, one must use the ```newData()``` func. In the initialized variable, one can add key-value pairs
```nim
var vars = newData()
vars["name"] = "Alice"
vars["age"] = 21

vars.set("weight", 50)

# or you can initialize it with a key-value pair
var vars = newData("name", "Alice").put("age", 21)
```
In a template, one must define the variables with matching names. Currently, if no variables are provided, the values will default to empty strings.
```html
<p>{[name]} is {[age]} years old.</p>
```

## Dynamic routes
To match a custom route and get the provided value(s), one must simply use a colon to specify a dynamic value. The values will be stored in the ```data``` parameter implicitly.
```nim
# User requests /countries/ireland/people/paddy
get "/countries/:country/people/:person": 
  assert(data["country"] == "ireland")
  assert(data["person"] == "paddy")
  respond tmplt("userPage", data)
)
```
```html
<h1>{[person]} is from {[country]}</h1>
```

## Subdomains
To add a subdomain to your application simply do the following:
```nim
subdomain "api":

  # Matches api.mysite.com
  get "/":
    respond %* {
      "status": "OK",
      "message": "Hello World!"
    }

  # Matches api.mysite.com/people
  get "/people":
    let people = @["adam", "beth", "charles", "david", "emma", "fiona"]
    respond newData("people", people)
```

## Hosts
Xander features the ```host``` macro, which makes it possible to run seperate applications depending on the ```hostname``` of the request header.
```nim
# Travel Blog
host "travel-blog.com":
  get "/":
    respond "Welcome to my travel blog!"

# Tech page
host "cool-techy-site.com":

  get "/":
    respond "Welcome to my cool techy site!"
  
  subdomain "api":
    get "/":
      respond %* {
        "status": "OK",
        "message": "Welcome"
      }
```

## TODO
- Add request headers to the ```headers``` variable
- Setting restrictions
- Error handling
- Code refactoring
- Web sockets
