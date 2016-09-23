+++
date = "2016-09-22T18:45:00+10:00"
draft = false
title = "Using the Learn REST API from Golang"
description = ""
keywords = ["blackboard", "REST", "API", "golang"]
+++

*By Shane Argo*

During the [DevCon at the Teaching & Learning Conference in Sydney](http://experience.blackboard.com/TLC-Sydney/pre-conference-events/devcon/) Wiley and I presented about Using the Learn REST API from Golang. This post is going to be a summary of that presentation.


## First things first - getting set up ##
First of all you'll need to [download and setup the development VM](https://community.blackboard.com/docs/DOC-1104). Be sure to download at least 2016Q2, as REST was unavailable before this. In fact, Blackboard is adding new REST services and improving the existing services in each version (even within the cumulative updates) so it's best to get the newest VM available.

Next you'll need to create an account on the [developer.blackboard.com site](https://developer.blackboard.com/). Once you've created an account you'll be invited to register an application. Click the register button and give your application a name and description. On the next screen you'll be presented with an 'Application Key' and 'Secret'. **Be sure to record these somewhere** as this is the one and only time you'll be given them. 

More information about [the Blackboard REST web services](https://community.blackboard.com/docs/DOC-1592) and [the developer portal](https://community.blackboard.com/docs/DOC-1579) is available on the community site.

Now that you've registered an application you can create a REST API Integration in Blackboard. From the System Admin tab in Blackboard, click on 'REST API Integrations' and then 'Create Integration'. You'll be asked for the Application ID you wrote down before. You'll also be asked for the Blackboard user you wish to run the Application as.

<center>![Let's just use the administra... No! - Batman slaps Robin](/images/batman-admin-account.jpg)</center>

The Learn REST APIs use [two-legged OAuth 2 with Client Credentials](http://oauthbible.com/#oauth-2-two-legged). Your application will access the REST API on Learn as the user specified while creating the integration in Blackboard. Please do not use the administrator user or role, instead create a new System Role with the required permissions and then grant that role to a for-purpose user. 


## cURL is your friend ##
Mark Kauffman from from Blackboard has posted a very good [blog post on the Blackboard community site about using cURL to experiment with the REST web services](https://community.blackboard.com/people/mkauffman/blog/2016/05/07/rest-with-curl). It is well worth the read, but to summarise:

Ask for a token:
````bash
curl --user <app_key>:<app_secret> --data "grant_type=client_credentials“ http://localhost:9876/learn/api/public/v1/oauth2/token
````

The service will respond with a token:
````JSON
{
   "access_token": "bQ8MZ3e2lFJAEwo2QhIJupbenhhREO0n",
   "token_type":"bearer", 
   "expires_in":297
}
````

You are now free to use this token to get data (until it expires):
````bash
curl -X GET -H "Authorization: Bearer bQ8MZ3e2lFJAEwo2QhIJupbenhhREO0n" http://localhost:9876/learn/api/public/v1/courses
````

## Where are the errors? ##
When something goes wrong in Blackboard the stdout/stderr log is usually the first place to look. This is not the case with the REST web services. Instead, look in the `/blackboard/logs/bb-services-log.txt` file.


## Get GOing ##
Alright, now let's look at setting up and using Go. Firstly, [you'll need to download Go](https://golang.org/dl/) for your system and [set it up according to the official documentation](https://golang.org/doc/install).

Now for the world's quickest introduction to Go. It is a C-like language, developed by Google. It is pretty safe (compared to C, C++, etc) because it is garbage collected, bounds checked, statically typed and allows no pointer arithmetic. Go is pseudo object-oriented, meaning basically that you can associate functions with structures, and it is [duck typed](https://en.wikipedia.org/wiki/Duck_typing). You can also use any [existing C library very easily](https://blog.golang.org/c-go-cgo). 


## The net/http package ##
Go even comes with a core package named `net/http` that provides both an HTTP client and server implementation.

Let's have a look at how to do an HTTP request using this package:
````go
response, err := http.Get("https://www.alltheducks.com/")
if err != nil {
   log.Fatal(err)
}
defer response.Body.Close()
_, err = io.Copy(os.Stdout, response.Body)
if err != nil {
   log.Fatal(err)
}
````

This code sample introduces a lot of concepts. On the first line we can see that Go functions can have multiple returns; the call to `http.Get` returns both an object we named `response` and and one we've named `err`. The first line also introduces the `:=` operator. This operator saves you having to define your variables ahead of time. It says, create a variable named `response` and `err` and assign the values returned from `http.Get`. Go infers the variables' types from the signature of `http.Get`.

On the second line, we see an extremely common pattern in Go code. On this line we check if the `http.Get` method returned an error. If it has, we log the error and exit on line 3. 

Onto the next line. On this line we are `defer`ing a call to `response.Body.Close`. This simply means that, when this block of code exits, no matter how it exits, call `response.Body.Close`. This is a nice mechanism for ensuring your code cleans up after itself.

Finally, we call the `io.Copy` method to copy the contents of the `response.Body` to `os.Stdout`. There are a couple of interesting things going on on this line. `io.Copy` returns two values. The first is the number of bytes written and the second is the error, if there was one. The underscore tells the compiler that we are aware that the number of bytes written is returned by `io.Copy` but we don't need it. This is necessary, as Go has a number of language features normally supplied by the IDE, such as unused variable detection. The only other thing to note about this line of code is that the `=` operator is used instead of `:=`. This is because no new variables have been introduced, we are reusing the `err` variable from above.


## The code is on Github ##
All of the code used in the post is available in a repository on Github. [We've set it up so that each step is in its own branch.](https://github.com/AllTheDucks/bb-rest-go-demo/branches) You are welcome to clone the project and try this for yourself at each step.


## The Oauth2 package ##
Go comes with an implementation of OAuth2 which makes [accessing the Blackboard REST web services super simple](https://github.com/AllTheDucks/bb-rest-go-demo/blob/1-minimal-ws-client/main.go):

1. Create a 'Client Credentials' configuration (remember from above, this is the auth type used by Bb).
2. Create a Client using this configuration.
3. Issue a request to the REST web service end point (of course, remembering to defer the clean up).
4. Do what you want with the response (in the example below we copy it to `Stdout`)

````go
func main() {
   
   conf := &clientcredentials.Config{
       ClientID:     "---- Application Key goes Here ----",
       ClientSecret: "---- Application Secret goes Here ----",
       Scopes:       []string{},
       TokenURL: "---- Your Bb server Root --- /learn/api/public/v1/oauth2/token",
   }

   client := conf.Client(oauth2.NoContext)

   resp, err := client.Get("---- Your Bb server Root --- /learn/api/public/v1/users")
   if err != nil {
       log.Fatal(err)
   }
   defer response.Body.Close()

   io.Copy(os.Stdout, resp.Body)

}
````

The amazing thing about this package, is that it is entirely handling the requesting and refreshing of OAuth tokens for you. You can simply focus on accessing the data you want.

With this code in a file named `main.go` and the key, secret and Bb server root substituted in, you should now be able to run it:

````bash
go run main.go
````


## The flag package ##
It is not a good idea to hard code your configuration into your application. Instead, [let's bring in this configuration data with command line flags](https://github.com/AllTheDucks/bb-rest-go-demo/blob/2-add-flags/main.go). Unsurprisingly, Go makes this easy too, with the `flag` package.

````go
import "flag"

...

var serverRoot string
var appKey string
var appSecret string
var tokenUrl string
var usersUrl string

func init() {
   flag.StringVar(&serverRoot, "serverRoot", "", "The base URL of the Bb Learn server. e.g. https://mybb.inst.edu.au")
   flag.StringVar(&appKey, "appKey", "", "The Application Key")
   flag.StringVar(&appSecret, "appSecret", "", "The Application Secret")

   flag.Parse()

   if serverRoot == "" || appKey == "" || appSecret == "" {
      flag.Usage()
      os.Exit(1)
   }
   tokenUrl = serverRoot + "/learn/api/public/v1/oauth2/token"
   usersUrl = serverRoot + "/learn/api/public/v1/users"
}
````

The only thing that might be confusing about this is the `&`s in front of the variable names in the call to `flag.StringVar`. This `&` passes a pointer to the variable to the `flag.StringVar` function. **Everything** in Go is passed by value. But we want Go to be able to write the value of the flag into the variable we've set up, so instead we pass a pointer to it.

Let's try it out:
````bash
go run main.go --serverRoot="http://localhost:9876/" --appKey="---- Application Key goes Here ----" --appSecret="---- Application Secret goes Here ----"
````


## Encapsulating Logic in Go ##
Things are going to start to get pretty messy if we don't start encapsulating some of the logic. Next we'll [make a service struct which requests the data from the OAuth client, decodes it into structs and returns them](https://github.com/AllTheDucks/bb-rest-go-demo/blob/3-print-courses-to-csv/main.go#L71). 

Something important here is the difference between names that start with an uppercase letter, and those that start with a lowercase letter. Go uses this a signal for the visibility of the field, struct or function. Those that start with an uppercase letter are exported/publicly visible and those with a lowercase are not.

Let's start with the structs that represent the JSON result.

````go
type CoursesResult struct {
   Courses []Course `json:"results"`
   Paging Paging `json:"paging"`
}

type Paging struct {
   NextPage string `json:"nextPage"`
}

type Course struct {
   Id string `json:"id"`
   Uuid string `json:"uuid"`
   ExternalId string `json:"externalId"`
   DataSourceId string `json:"dataSourceId"`
   CourseId string `json:"courseId"`
   Name string `json:"name"`
}
````

The magic here is the strange stuff inside of the back ticks. These are called tags in Go and they allow you to specify some meta-data about the fields. They are somewhat akin to annotations in Java. Here were are using them specify how the JSON library should map the fields in JSON data.

And now to create a service struct and which is responsible for requesting the data and decoding it:

````go
type CourseService struct {
   Client http.Client
}

func (svc CourseService) getCourses() (coursesResult CoursesResult, err error) {
   resp, err := svc.Client.Get(coursesUrl)
   if err != nil {
      log.Fatal(err)
   }
   err = json.NewDecoder(resp.Body).Decode(&coursesResult)
   return
}
````

The first thing we did here was define a struct named `CourseService` with a single field named `Client` of type `http.Client`. The next thing we did was attach a function to the struct which is known as the receiver. 

It is often helpful to think of the receiver as just another function parameter. The reason it is useful to think of it this way is, as has already been mentioned, **everything** in Go is passed by value. This means that when the `getCourses` function is called, a copy of the `CourseService` is copied to the top of the stack, and thrown away again when returning. The result of this is that making changes to the struct inside this function will modify the copy on the stack.

Quite often this is not the expected behaviour especially if you are used to languages like Java. Remember earlier when calling `flag.StringVar` we passed in a pointer to the variable instead so that the function would put the value of the flag into the variable itself. If you think of the receiver as just another parameter this is easier to understand. You may want the function to modify the instance of the struct on which it was called, not a copy on the stack. So, as with `flag.StringVar` you pass a pointer or attach the function to a pointer.

In this case, it's not a problem, as we do not change anything on the `CourseService`, so we are going to leave it attached directly to the struct.

The only other new thing here is the `json` package. Here we construct a new decoder from the response body and then immediately use it to map the value into the `CourseResult` being returned.


## Calling our new service ##
Okay, [we're going to want use our new service struct](https://github.com/AllTheDucks/bb-rest-go-demo/blob/3-print-courses-to-csv/main.go#L52):

````go
client := conf.Client(oauth2.NoContext)

courseService := CourseService{Client: *client}
courses, err := courseService.getCourses()

if err != nil {
   log.Fatal(err)
}

fmt.Printf("Id, ExternalId, CourseId, Name\n")
for _, c := range courses.Courses {
   fmt.Printf("\"%s\",\"%s\",\"%s\",\"%s\"\n", c.Id, c.ExternalId, c.CourseId, c.Name)
}
````

In this code, we create an HTTP client, setup a `CourseService` struct passing it the client, and then use the service to get the courses. Obviously, as we've done many times now, we check if there was an error and, assuming there wasn't, we use the result.

The next few lines of code are not very robust, but serve well as a demonstration. The results really should be properly escaped. This aside, we print a CSV header row, and then print out a row per result.


## OMG, an HTTP server ##
In Java you write an application that runs within an HTTP container, such as tomcat. In Go you write an application that contains an HTTP server. As I mentioned earlier, the net/http package has a server implementation, which we'll use to [serve the CSV data](https://github.com/AllTheDucks/bb-rest-go-demo/blob/4-serve-csv-file/main.go).

The first thing we'll do is set up a handler function which is called to handle a request:
````go
func courseListHandler(w http.ResponseWriter, r *http.Request) {
   courses, err := courseService.getCourses()

   if err != nil {
      log.Fatal(err)
   }
   w.Header()["Content-Type"] = []string{"text/csv"}
   w.Header()["Content-Disposition"] = []string{"attachment; filename=\"courselist.csv\""}
   fmt.Fprintf(w, "Id, ExternalId, CourseId, Name\n")
   for _, c := range courses.Courses {
      fmt.Fprintf(w, "\"%s\",\"%s\",\"%s\",\"%s\"\n", c.Id, c.ExternalId, c.CourseId, c.Name)
   }
}
````

This is all pretty self explanatory. The only differences between this and the code above is that we're now specifying a few headers for the response and we're writing the CSV data to the response instead of to stdout. Next we'll set up the web server:

````go
func main() {
   ...
   http.HandleFunc("/", courseListHandler)
   http.ListenAndServe(":8080", nil)
   ...
}
````

The most interesting thing in this code is the passing of a function as an argument. Our handler function implements the interface expected by the `http.HandleFunc` function and can therefore be passed in directly.

## What next? ##
In the presentation at the conference, we take this project one step further and serve some templated HTML which lists the courses with each being a link to a path that downloads a CSV of the enrolments. In this post, I've [left it as an exercise for the reader to dig around in Github and check out](https://github.com/AllTheDucks/bb-rest-go-demo/tree/5-template-html).

