---
title: Visual explanation of SAML authentication
description: High level explanation of the SAML authentication protocol for beginners
keywords: SAML, SSO, Enterprise Software, Authentication
image: /images/2020-saml/image-3.png
date: 2020-06-19 07:31:25
tags:
  - SAML
  - Authentication
---

SAML (Security Assertion Markup Language) is the most commonly used authentication protocol and SSO solution in enterprises.

![](/images/2020-saml/image-1.png)

## What is SSO?

To put it simply, it's the enterprise equivalent of the "Login with Google" or "Login with Facebook" buttons we see in apps around the internet. We register an account initially in Google or Facebook etc and use that account to login to other apps like Spotify, Netflix, Zoom etc. We do this to avoid maintaining multiple username/passwords. Similarly, enterprises maintain a single user management system and employees use their corporate account to login to third-party services like Salesforce, Workday, Expensify etc without creating separate accounts or remembering multiple passwords. This is called SSO (Single Sign On) and SAML is the de facto enterprise SSO solution.

## Participants

There are 3 main participants involved in the SAML authentication flow:

### Identity Provider (IdP)

This is the centralised user management system that we talked about earlier. This server is responsible for authenticating the user and passing the user details such as email address, name, department etc to the Service Provider. Popular identity providers are Azure AD, Auth0, Onelogin, Okta, G Suite etc.

### Service Provider (SP)

This is the application that trusts the IdP and wants to use it for authentication. Examples: Salesforce, Workday, Expensify, \$YOUR_AWESOME_APP etc

### Principal

This is the user who's trying to log into the SP via the IdP.

## Authentication Flows

There are two common ways for an user to access SP:

### IdP initiated login:

The user goes to the IdP first and is shown a list of SP they have access to. Upon choosing an SP from that list, they're redirected to that SP.

### SP initiated login:

In this flow, the user goes to the SP's website first. If the user doesn't have an active session with the SP, the user is redirected to the IdP for authentication. Upon successful login, the user is redirected back to the SP. We'll be discussing this flow in detail.

## SP initiated Flow:

Let's talk about the flow from the user's perspective.

- The user goes to the SP's website. If the user is not logged in, it shows a "Login with SSO" button
- Upon clicking the login button, the user is redirected to the IdP's website where they're asked to submit their credentials
- Upon successful login, the user is redirected back to the SP's website where they can perform their work

![](/images/2020-saml/image-1.png)

Now, let's zoom in a bit and understand what happens behind the scenes:

- SP checks for active session
- SP sends AuthnRequest to IdP
- IdP authenticates the user
- IdP sends SAML Assertion to SP
- SP creates session and logs in user

### SP checks for active session

SAML doesn't maintain sessions, so SP needs to maintain sessions for each authenticated user. When a user visits the SP website, it checks whether the user has an active session with it. If an active session exists, the user can enter the website otherwise a "Login with SSO" button is shown.

![](/images/2020-saml/image-2.png)

### SP sends AuthRequest to IdP

When the user clicks on the "Login with SSO" button, the SP generates a XML message called "AuthnRequest" with details about who's sending the request (Issuer), where to redirect to after the user is authenticated (Assertion Consumer Service url) and security measures (ID, IssueInstant). Here's an [example AuthnRequest XML](https://www.samltool.com/generic_sso_req.php).

This XML is encoded into a url-safe string, embedded as query param in a request to IdP and the user is redirected to this IdP url:

`https://idp.com/SAML2/SSO/Redirect?SAMLRequest=EncodedAuthnRequest`

### IdP authenticates the user

IdP maintains its own session about the user and if an active session exists for the user, the user is redirected to SP. If a session doesn't exist, the user is asked to enter their credentials. The IdP can choose how to authenticate the user - can be Username/Password, TOTP, MFA etc.

### IdP sends SAML Assertion to SP

Once the user is successfully authenticated, IdP sends back an XML message called "SAML Assertion" to the SP's Assertion Consumer Service url. This contains the user's details such as name, email, department etc and security measures (InResponseTo, IssueInstant). It's also digitally signed so the SP can trust that the message is indeed from IdP and login the user into their system.

![](/images/2020-saml/image-3.png)

### SP creates session and logs in user

The user is now successfully logged in to the SP's website! The SP will create a session for the user so the user can be automatically logged in the next time they visit the website.

## Conclusion

Hopefully this post is able to give you a high level overview of SAML authentication and how the SSO works.

Thanks for reading! :)
