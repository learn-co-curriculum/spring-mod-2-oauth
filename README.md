# OAuth

## Learning Goals

- Define OAuth and explain how it works.

## Introduction

In the last lesson, we learned about authorization and how we could authorize a
user by checking for certain permissions once in our application. To do this, we
used HTTP basic to authorize the user - which is checking the user by its
username, password, and the authorities the user holds.

But what if we didn't always want to provide username and password credentials?
For example, say we downloaded a new application, and it gives us the option to
login with Facebook instead of creating an account. If we choose that option, it
may take us to Facebook where we can say we will allow this app permission to
see some information gathered from the Facebook profile. After agreeing, it will
direct us to the new app we downloaded.

So what happened here? Did Facebook share our username and password credentials
with the application?

## What is OAuth?

"**OAuth**, short for Open Authorization, is a standard designed to allow an
application to access resources hosted by other web apps on behalf of a user in
order to provide consented access and restricted actions of what the client app
can perform without ever sharing the user's credentials"
([What is OAuth 2.0?](https://auth0.com/intro-to-iam/what-is-oauth-2)).

This is the exact authorization process that happened in the example scenario
described above: We were able to access the new application without having to
create an account by linking it to our Facebook, allowing that application to
access resources hosted by Facebook on behalf of the user in order to provide
consented access and restricted actions.

So back to the question, did Facebook share our username and password with the
application?

OAuth does **not** share passwords or sensitive information. Instead, OAuth will
work with something called an **access token**. Access tokens are generated and
provided to access resources, which is then passed to the application to grant
the appropriate permissions. These tokens come in a specific format, usually as
a **JSON Web Token** or JWT.

## How Does OAuth Work?

Let's look at the process of how OAuth may work. Assume we have an application,
and we are going to use GitHub as our OAuth provider. This means, in order to be
authorized to our application, we'll need to link our application to our GitHub
account so that it can provide the application a token, allowing us into the
application.

Consider the following diagram:

![Spring OAuth Flow](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-oauth-flow.png)

Before we explain the entire process, let's define the following:

**Resource Owner**: This is typically the end-user or another server
requesting authorization.

**Client**: This is the application that the resource owner interfaces with and
performs the protected requests made.

**Authorization Server**: This is responsible for generating an access token
after authenticating the resource owner.

**Resource Server**: This is responsible for providing whatever protected
resource is being requested.

In the diagram above...

1. The resource owner and end-user are the same since it is a human-being making
   the request out to the client.
2. Once the request is sent to the application, the application will come back
   asking the user to grant permission to interact with GitHub on their behalf -
   that's what the Authorization Grant represents.
3. The application then automatically sends the Authorization Grant to the
   authorization server (or GitHub in this example).
4. The authorization server then redirects back to the client with an
   authorization code, letting the application know that the user may be
   authorized with a generated access token.
5. In order for the user to be authorized on the client side, it makes a request
   out to the resource server with the access token to access specific
   resources.
6. The resource server will then provide the application with the appropriate
   user information to fully have the user authorized.

This is the same process if we were to download an app and login using
Facebook or Google!

OAuth is considered a standard in the industry when it comes to authorization
since it does not share sensitive information, like a password, making it more
secure.

## OAuth in Spring

We can implement OAuth using the Spring Framework as well! To do this, you will
need to add the "Oauth2 Resource Server" dependency to the `pom.xml` file.

In this lesson, we will not cover how to exactly implement this feature, but if
curious, please see the Spring documentation along with these other helpful
tutorials:

- [OAuth 2.0 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html)
- [Djamware: Spring Boot, Security, PostgreSQL, and Keycloak REST API OAuth2](https://www.djamware.com/post/6225b66ba88c55c95abca0b6/spring-boot-security-postgresql-and-keycloak-rest-api-oauth2#create-spring-boot)
- [[Video] Laur Spilca: Spring Security Fundamentals - Lesson 11 - The OAuth Authorization Server](https://youtu.be/N0LiMIGCDgg)

## References

- [What is OAuth 2.0?](https://auth0.com/intro-to-iam/what-is-oauth-2)
- [Introduction to API Gateway OAuth 2.0 Server](https://docs.oracle.com/cd/E55956_01/doc.11123/oauth_guide/content/oauth_intro.html#:~:text=An%20entity%20capable%20of%20granting,resource%20requests%20using%20access%20tokens.)
- [The Difference Between Basic Auth and OAuth](https://squareball.co/blog/the-difference-between-basic-auth-and-oauth)
