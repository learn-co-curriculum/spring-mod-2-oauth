# OAuth

## Learning Goals

- Define OAuth
- Implement OAuth

## Introduction

OAuth is an open standard that allows a user, to grant an application, such as
the one we're building here, access to the user's information that is stored and
managed by a third party.

For example, a user might want to allow your application to use their Google
account information in order to validate who they are and give them permissions
to perform specific actions. In this scenario, and in OAuth parlance, we have
the following components:

1. The user is the "Resource Owner", i.e. the person whose credentials are being
   validated. The "credentials" are the resource being owned here.
2. Google is both the "Authorization Server" and the "Resource Server".
   1. The "Authentication Server" is responsible for generating an "Access
      Token" once it has validated that you are who you say you are
   2. The "Resource Server" is responsible for providing whatever protected
      resource is being requested
3. The "Client" is the application that is requesting access to the protected
   resource.

## OAuth in Spring

The Spring Framework provides support for OAuth, so let's explore this mechanism
more by implementing it for our application.

Since we're about to add static content to our sample application, let's first
clean up our existing URLs to make an explicit distinction between our
"endpoints" and our "content". We'll do this by moving all our existing
endpoints to a "/api/" base URL:

```java
    @GetMapping("/api/hello")
    public String hello(@RequestParam(name = "targetName", defaultValue = "Stephanie") String name) {
        String greeting = "Hello " + name;
        greeting += "<br/>";
        greeting += "Dad joke of the moment: " + jokeService.getDadJoke();
        return greeting;
    }

    @GetMapping("/api/status")
    public String status() {
        return "Congratulations - you must be an admin since you can see the application's status information";
    }
```

This means you have to update all your integration and acceptance tests to make
them use the updated URLs - you should be able to do that on your own.

We also need to update our security configuration to only require authentication
for the API-based URLs, as we want our static content to be accessible to
anyone, at least for now:

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/api/status")
            .hasAuthority("admin");

        http.authorizeRequests()
            .antMatchers("/api/**")
            .authenticated()
            .and()
            .formLogin()
            .and()
            .logout();
    }
```

## Simple UI

Now let's add a simple `index.html` page to our `src/main/resources/static`
project folder:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Demo</title>
    <meta name="description" content="" />
    <meta name="viewport" content="width=device-width" />
    <base href="/" />
    <link
      rel="stylesheet"
      type="text/css"
      href="/webjars/bootstrap/css/bootstrap.min.css"
    />
    <script type="text/javascript" src="/webjars/jquery/jquery.min.js"></script>
    <script
      type="text/javascript"
      src="/webjars/bootstrap/js/bootstrap.min.js"
    ></script>
  </head>
  <body>
    <h1>Demo</h1>
    <div class="container"></div>
  </body>
</html>
```

Let's make sure Bootstrap and jQuery are available by adding the corresponding
dependencies to our project:

- Gradle:

```json
	implementation 'org.webjars:webjars-locator-core'
	implementation 'org.webjars:jquery:3.6.0'
	implementation 'org.webjars:bootstrap:5.1.3'
```

- Maven:

```xml
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>jquery</artifactId>
	<version>3.4.1</version>
</dependency>
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>bootstrap</artifactId>
	<version>4.3.1</version>
</dependency>
<dependency>
	<groupId>org.webjars</groupId>
	<artifactId>webjars-locator-core</artifactId>
</dependency>
```

## Using GitHub as the OAuth Provider

First, we need to add Spring's OAuth support to our project by adding the
following dependencies:

- Gradle

```json
	implementation 'org.springframework.security:spring-security-oauth2-resource-server'
	implementation 'org.springframework.security:spring-security-oauth2-jose'
	implementation 'org.springframework.security:spring-security-oauth2-client'
```

- Maven

```xml
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-oauth2-resource-server</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-oauth2-jose</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-oauth2-client</artifactId>
</dependency>
```

There are many OAuth providers - since we can be sure every single one of you
has a GitHub account, we will use GH as the OAuth provider.

Log in to your GitHub account and go to
`https://github.com/settings/developers`:

![GH Dev Settings](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-gh-dev-settings.png)

Click on the button to "Register a new application" and fill out the following
details:

![GH OAuth New Application](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-gh-new-application.png)

We are using all default values for your application settings:

1. Homepage URL: is the base URL for your application - your port number might
   be different if you've customized your application settings
2. Authorization callback URL: we're using the default value here as well, so
   adjust accordingly if you've updated the defaults

Registering the application will take you to a screen that confirms that the
application has been registered and shows you your Client ID, which has been
blurred here for obvious reasons:

![GH OAuth Client ID](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-gh-client-id.png)

Select the option to generate a "Client Secret" as well.

You should have 2 values on your GH Developer Settings for the "Flatiron Spring
Security Training" application you just created:

1. Client ID: this identifies your application so that the OAuth provider
   (GitHub in this case) can look up the settings associated with your
   application when it (your application) requests access on behalf of the user
2. Client Secret: this is a "password" of sorts that only your client
   application should know, so that it can prove to GH that it is authorized to
   make the request it's making

Note: in this simple sample application, we are storing the client secret in our
source code. This should generally not be the case for production application
because many more people have access to your source code (especially when you
work within large organizations) than are supposed to have access to your
"secrets". All cloud providers have "secret managers" that let applications
manage their secrets for different environments.

Make sure to copy your client secret in a safe place where you can use it in a
moment when we configure the application. Once you refresh this page, your
client secret will no longer be visible (for security reasons) and you will not
be able to retrieve it.

![GH OAuth Client Secret](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-gh-client-secret.png)

Now we have to change our security configuration to ask the static content to be
authenticated with OAuth2:

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/api/status")
            .hasAuthority("admin");

        http.authorizeRequests()
            .antMatchers("/api/**")
            .authenticated()
            .and()
            .formLogin()
            .and()
            .logout();

        // adding a rule to require authentication for all content via OAuth
        http.authorizeRequests()
            .anyRequest()
            .authenticated()
            .and()
            .oauth2Login();
    }
```

If you restart your application now and try to access your "index.html", you
will notice that you have both `httpBasic` and `oauth` login enabled:

![GH OAuth and HTTP Basic](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-gh-and-http-basic.png)

This is because we have mixed both authentication methods in our security
configuration. Let's clean that up by setting our API access to also use OAuth
authentication:

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/api/status")
            .hasAuthority("admin");

        http.authorizeRequests()
            .antMatchers("/api/**")
            .authenticated()
            .and()
            .oauth2Login() // change httpBasic() to oauth2Login() for API resources
            .and()
            .logout();

        http.authorizeRequests()
            .anyRequest()
            .authenticated()
            .and()
            .oauth2Login();
    }
```

You should now be redirected to GitHub for all login requests.

Let's summarize the high level flow of operations in order for the OAuth
provider, in this case GitHub, to provide the user information to the client, in
this case your application:

![Spring OAuth Flow](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-flow.png)

1. The security configuration in our application results in our application
   sending an Authorization Request to GitHub whenever the user tries to access
   a restricted resource, which in our case is any resource
2. Our user needs to grant our application permission to interact with the OAuth
   Provider (GitHub) on their behalf - that's what the Authorization Grant
   represents
3. The Spring Framework OAuth support then automatically sends the OAuth
   Provider the Authorization Grant, which GH then validates
4. Upon Authorization Grant validation, GH returns an Access Token, which can
   then be used to request specific resources from the OAuth Provider
5. The client (your application) can then use the Access Token to request
   resources from the OAuth Provider. In our case, the Spring Framework OAuth
   support requests user information from GH
6. GH returns user information and we now have an authenticated user in our
   application context

## Conclusion

We have learned about OAuth and added GitHub OAuth to the project. It is usually
better to use an external auth service since keeping up with latest security
patches and maintaining your own auth service can take up a lot of resources.
