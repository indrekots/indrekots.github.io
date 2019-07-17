---
layout: post
title: "How to Invalidate JSON Web Tokens"
excerpt: "JWTs allow us to implement stateless authentication. We don't have to query a central database to validate the token. But how do we invalidate them?"
modified: 2019-07-17 18:56:18 +0300
categories: articles
tags: [JWT, JSON Web Token, authentication]
image:
  path: /images/2019-07-17-invalidate-jwt/cover.jpg
  thumbnail: /images/2019-07-17-invalidate-jwt/cover_thumb.jpg
  caption: "Photo by [ZSun Fu](https://unsplash.com/photos/b4D7FKAghoE)"
comments: true
share: true
published: true
aging: false
---

The most common use case for [JSON Web Tokens](https://jwt.io/introduction/ "Introduction to JSON Web Tokens") (**JWT**) is authorization.
Once a user has provided his/her credentials, the server issues a JWT that the user will have to include in each subsequent request.
[The benefit of JWTs is that they're stateless](https://www.jbspeakr.cc/purpose-jwt-stateless-authentication/ "The Purpose of JWT: Stateless Authentication").
We don't have to query a central database to validate the token.
As long as the signature is correct and the token hasn't expired, we can trust it and allow the user to access the restricted resource.
This is good when you wish to reduce the load on your database but it makes invalidating an existing non-expired token difficult.
Let's say a user has logged out, how do we make sure the token cannot be used anymore?

## Storing tokens in a database

The most obvious approach would be to store the token in a database.
We can check which tokens are valid and which ones have been revoked.
But this defeats the entire purpose of using JWTs in my opinion.
Essentially, we would be keeping a list of all issued tokens, making them stateful.
There *has to be* a central authentication manager that would have to check every incoming request against the list of stored tokens.

## Delete token from the client

When a user logs out, the client app should delete the token from its memory.
This would stop the client from being able to make authenticated requests.
But if the token is still valid and somebody else has access to it, the token could still be used.

## Short token lifetime

Let the tokens expire quickly.
Depending on the application, it could be several minutes or half an hour.
When the client deletes its token, there's a short window of time where it can still be used.
Deleting the token from the client and having short token lifetimes would not require major modifications on the server side. But short token lifetimes would mean that the user is constantly being logged out because the token has expired.

## Rotate tokens

It is possible to introduce a concept of refresh tokens.
When the user logs in, we can provide them with a JWT and a refresh token.
The refresh token will be stored in a database.
For authenticated requests, the client can use the JWT but when the token expires (or is about to expire), let the client make a request with the refresh token in exchange for a new JWT.
This way you would only have to hit the database when a user logs in or asks for a new JWT.
When the user logs out, you would need to invalidate the stored refresh token.
Otherwise somebody listening in on the connection could still get new JWTs even though the user had logged out.

## Create a JWT blacklist.

Depending on the expiration time, when the client deletes its token, it might still be valid for some time.
If the token lifetime is short, it might not be an issue, but if you still wish that the token is invalidated immediately, you could create a token blacklist.
When the server receives a logout request, take the JWT from the request and store it in an in-memory database.
For each authenticated request you would need to check your in-memory database to see if the token has been invalidated.
To keep the search space small, you could remove tokens from the blacklist which have already expired.

## Summary

In its core, JWTs cannot be edited once they have been issued.
If you wish to invalidate them, you have to start keeping some state.
Ask yourself, why are you using JWTs.
If you really need to invalidate them, perhaps using regular stateful authentication is a simpler approach?
