# Single Sign-On(SSO) 

## [Single Sign-On (SSO) Simplified: Understanding How SSO Works in Plain English](https://medium.com/p/7d5739d23aeb)


## Background

Do you remember back in the days when you had to remember multiple usernames and passwords for all your online accounts? Accounts for utility bills, phone bills, social media, shopping, online booking for physiotherapist, etc.. Well, I certainly do! I was so tired of trying to manage all those passwords…

What about now? Well, lots of websites and mobile applications support “Sign in with Google”, “Sign in with Facebook” or “Sign in with Twitter”, etc., which has made life a lot easier.

This is the same for enterprises as well. Many companies have implemented a Single Sign-On (SSO) system, which allow their employees to access the company’s resources like websites and apps with just one set of login credentials.

Over the past few years, I’ve got a chance to implement SSO for a few different enterprises’ software systems. Every time when the product was delivered, there was that “aha” moment from them!

## Goal

Discuss how single sign on (SSO) works and how to implement SSO in an organization at a high-level.

## What is Single Sign-On

Single Sign-On (SSO) is a method of authentication that allows users to access multiple applications and systems with one set of login credentials (username and password). The user only needs to enter their login information once, and they will be automatically logged in to all of the systems and applications that they have been granted access to.

* Benefits

Obviously, the two major benefits are:

1. User convenience as users only need to remember one set of credentials
2. Improved security by reducing the number of places where user passwords are stored

Before we look at how SSO works for software applications, let’s first recap how a similar concept works for a hotel guest.

In the diagram above, the hotel guest goes through the following steps:

1. Check in by showing passport or other type of identification
2. Get a hotel card, which grants the guest access to a private room, the gym open to all guests, and optionally valet parking code, i.e., #G372B
3. Tap room door smart lock, which checks whether you are a hotel guest AND whether you should have access to this room), and go sleeping
4. Tap swimming pool door smart lock, which checks whether you are a hotel guest, and go swimming
5. Call valet parking service to bring the car to the front by providing the valet parking code (no hotel card needed, just the code)
6. Check out (return or destroy the hotel card)

Notice the guest only needs to show the identification once, and will be able to use different resources. Pretty straightforward, right? Well, SSO in software systems work in a very similar way. Let’s modify the diagram above slightly. Note in the diagram below, the hotel guest becomes a user, the front desk becomes our SSO server web application, the hotel room becomes a server-side web application called Room web app, the swimming pool becomes a SPA application called Swimming pool app, and the valet parking service becomes a web API service called Valet Parking web API.

## How does Single Sign-On work

In the diagram above, a user can go through the following steps:

1. User login to the SSO server web application by providing their email and password. This is called authentication, and is like the hotel check-in step.

2. SSO server web application validates the user credentials and issues a local cookie, name it cookie1, which allows the user to remain signed into the SSO server web application. This is like the front desk issuing a hotel card.

3. User attempts to access the Room web app for the first time. Under the hood, the user will be redirected to the SSO server web app, which knows that the user is already logged in from cookie1 and provides the user identity data (i.e., user first name, last name, etc.) and access data (i.e., room number, and optionally valet parking API access code). The Room web app is a server-side web application and will issue its own local cookie, name it cookie2, which contains the identity data and access data above. VERY IMPORTANT, note we purposely call it out that SSO server web app issues cookie1 and the Room web app issues cookie2, because they are cookies for different domains. Cookie1 will only be included in the requests sent to SSO web app, and cookie2 will only be included in the requests sent to the Room web app. At this point, two cookies should have been issued. This step is very similar to the hotel guest accessing their room.

4. User attempts to access the Swimming pool app that is open to all hotel guests. Similarly to the step above, under the hood, the user will be redirected to the SSO server web app, which knows that the user is already logged in from cookie1 and returns the user identity data (i.e., user first name, last name, etc.) and access data (i.e., valet parking API access code). No room number is needed. Because the Swimming pool app is a client-side SPA, it will store the identity data and access data in the browser, such as session storage or local storage.

5. User attempts to call the valet parking service. In fact, the Room web app or the Swimming pool app will call the Valet parking web API on behalf of the user, by attaching the access code in the request to the Valet parking web API. For additional details, this access code is often an access token like JWT token attached in the “Authorization” header in the HTTP request.

6. User logout of all the applications. This step will clear cookie1, cookie2, and the data stored on the SPA client-side. Similar to the hotel checkout step, the hotel guest should lose access to all resources.

This is how SSO works at a very high-level with minimal technical details. Next, let’s discuss how to implement SSO in your own organization at the high level.

## How to build a SSO server

Looking at the diagram above, we will need at least the following components to get started.

1. User management. This allows us to manage the usernames, passwords, emails, etc..

2. Configuration data management. This is to manage what applications should have access to the SSO server, how they should communicate, and what resources these applications should have access to.

3. Operational data management. This is to manage data generated by the SSO server, such as refresh tokens.

4. Keys for digital signatures. We mentioned above that JWT token is a common access token. JWT token needs to be digitally signed by such keys.

5. SSO protocol implementation. There is an industry standard regarding how SSO should work, so we should follow it. The standard is called OpenID Connect and OAuth2. OpenID Connect is for authentication purpose and is built above OAuth2, which is for authorization.

Typically, in order to implement SSO in a .NET Core web application, the following libraries and persistence stores can be used on top of the .NET Core framework:

1. .NET Core Identity for user management. This handles everything about the user, such as user profile, passwords, roles, password resets, email confirmation tokens, etc..

2. Identity Server, as a framework recommended by Microsoft documentation, and it implements the OpenID Connect and OAuth 2.0 protocols and helps manage the configuration data and operational data.

3. SQL database such as SQL Server, Azure SQL. The user data, configuration data, and operational data is highly relational, and updates on this data should be reliable and transactional. Plus, the data size is small for fit into one single database and horizontal sharding is unlikely required. A relational SQL database would be a good fit.

## Conclusion

In this article, we briefed discussed what single sign-on (SSO) is and how SSO works in laymen’s terms at the high-level. Hopefully next time when you have to explain SSO to a non-technical manager or executive, you will have a reference!

## [Build Your Own Authentication Server for Single Sign-On (SSO) in ASP.NET Core](https://medium.com/geekculture/build-your-own-single-sign-on-sso-server-in-asp-net-core-4344f6b390d1)

## Getting Started

### System Diagram

Let’s first look at the system diagram below. The SSO Server is single web application, and it communicates with a single database. Tables for user data, configuration data, and operational data can all be sitting in the same database.

The most obvious advantage of having configuration data and operational data stored in a database instead of code files or app settings files, is that changes to such data will not requires app restart. Another benefit is that we can run multiple instances of the SSO server in a load balancing environment. These instances all share the same data.

### Dry Run

You should be able to just follow along without the source code. However, if you would like to navigate through the code or even give it a dry run, the sample code in this article is hosted in a [GitHub project](https://github.com/ShawnShiSS/single-sign-on). You can simply clone the repo, and run the SSO Server project in Visual Studio or using command line. See more details in the README file.
