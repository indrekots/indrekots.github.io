---
layout: post
title: "Consumer Driven Contract Testing with Pact"
excerpt: "Consumer driven contract testing is a method of verifying that services speak the same language. It is an alternative to traditional integration testing that gives you faster feedback."
modified: 2019-05-05 18:56:18 +0300
categories: articles
tags: [pact, cdct, testing]
image:
  path: /images/2019-02-25-consumer-driven-contract-testing/cover.jpg
  thumbnail: /images/2019-02-25-consumer-driven-contract-testing/cover_thumb.jpg
  caption: "Photo by [Photo by rawpixel](https://unsplash.com/photos/YqwOX6Ks9k8)"
comments: true
share: true
published: false
aging: false
---

In the previous post, we had a look at [different ways of testing microservices]({{site.url}}/articles/challenges-of-testing-microservices/ "The Challenges of Testing Microservices") and learned that *classical* testing methods might not work well in distributed environments.

To recap, testing services in isolation with mocks is fast and cheap but not reliable.
Once we change the API of a service, we have to make sure we update the tests in all of its clients.
Essentially, service mocks only assume how the real counterpart should behave and the only thing that verifies this assumption is us.
Therefore it's very easy for mistakes to creep in.

Spinning up two services to test the integration between them seems like the next logical step.
As we saw in [the previous post](({{site.url}}/articles/challenges-of-testing-microservices/ "The Challenges of Testing Microservices"), this approach is definitely more reliable compared to testing with mocks but it's also slower and takes more effort to maintain.
A service can have many clients, increasing the time it takes to verify a single change.
In addition, if we want to deploy our services independently of others, we have to test for backwards compatibility with services already deployed to production.
The same is true if we were to spin up all of our services and performed end-to-end testing.
Lots moving parts can make our tests flaky and we run the risk of becoming a victim of [*normalization of deviance*](https://en.wikibooks.org/wiki/Professionalism/Diane_Vaughan_and_the_normalization_of_deviance "Professionalism/Diane Vaughan and the normalization of deviance").

## Consumer Driven Contract Testing

*Consumer driven contract testing* is a method of verifying that services (e.g. API consumer and an API provider) speak the same language.
By providing examples, API consumers set expectations on providers on how they should behave on specific inputs.
A set of expectations forms a contract that's produced by consumers and is shared with providers.

Contract obligations are verified by providers with tests that can be run in isolation, without having to set up integration testing environments.
That lets them evolve independently and get immediate feedback when they've broken any of their API consumers.
Contract testing can be used anywhere where you have two services that need to communicate with each other but becomes especially useful in environments with many services (e.g. microservice architecture).

## Pact

Pact is a consumer driven contract testing tool originally written by a development team at realestate.com.au.
Essentially, two services enter into contract on how to communicate with each other and Pact verifies whether both sides honor the agreement.
In Pact terminology, the contract is referred to as a *pact*.

What makes it *consumer-driven* is the fact that a client of an API (e.g. Alice's Billing service) sets expectations on the provider of the API (e.g. Bob's Customer service) on how to behave.
Expectations are set by examples.
For instance, if a GET request is sent to `/customers/17`, the Customer service should respond with HTTP 200 and with the customer data belonging to the given customer.
A collection of these interactions are encoded into a JSON document called a pact file.

Now that the interactions between Billing and Customer service have been defined, the pact file can be used to check whether both sides of the contract behave as agreed upon.
To do that, Alice has to write some tests that exercise Billing service's code.
But instead of sending requests to a real Customer service (i.e. an integration test), Pact starts up a mock HTTP server that intercepts all requests and records them.
If a request matches to a request in the pact file, the mock server will respond with the response encoded in the pact file.
But if the incoming request is not in the pact file, the test will fail, indicating that the consumer (in this case Alice's Billing service) does something that was not defined in the contract.

When the consumer of an API has defined a pact and ensured that it complies with it, it is time for the provider side of an API to verify it.
Bob received a pact file from Alice and immediately started to modify Customer service's build process.
He included a pact verification step that starts up a Pact mock HTTP server and the Customer service.
Pact's mock server will take all the requests from the pact file and play them against a running instance of a Customer service.
Then it observes how Customer service responds and compares the responses to the ones in the pact file.
If they don't match, the test will fail and Customer service is deemed not compatible with Billing service.

Looking at the bigger picture, Pact allowed Alice and Bob to verify whether Customer and Billing service speak the same language without having to spin up both services and writing classical integration tests.

## Sharing pacts

After consumer build has generated a pact file, the provider should have access to it in order to verify the pact.
Pact files can be [shared in multiple ways](https://docs.pact.io/getting_started/sharing_pacts#alternative-approaches).

### Store pact files in VCS repository

If the consumer and provider live in the same repository, pact files could be stored in the same repository.
Once the consumer CI build has finished, it could commit the generated pact file into the same repository.

### Consumer CI build commits pact file to provider codebase

This is relatively similar to the previous option.
Once your consumer CI build has generated the pact file, add an extra step that commits the file to the provider's repository.

### Consumer CI build publishes pact files as build artifacts

In addition to storing the consumer application as a build artifact in an artifact repository, the CI build could also publish pacts as build artifacts.
On the provider side, you would need to figure out how to construct the URL that points to the latest pact file in your artifact repository to access it.

### Store pacts on a network file share or AWS S3

Consumer CI build could store generated pacts on a shared storage that's accessible to the provider build.
You could use your in-house system of upload pacts to AWS S3 for example.
[Retreaty](https://github.com/fairfaxmedia/pact-retreaty "Easily share pacts via S3") is a Ruby gem that provides a ultra light mechanism for pushing these contracts to S3 from a consumer, and later pulling them down to a provider for verification.

## Pact Broker

The recommended way to share pacts is to use the [Pact Broker](https://github.com/pact-foundation/pact_broker).
Although, sharing pacts via AWS S3 or VCS repositories gets the job done, you are generally only verifying that the head versions of your services play nice with each other.
To be able to [deploy services independently](https://www.rea-group.com/blog/enter-the-pact-matrix-or-how-to-decouple-the-release-cycles-of-your-microservices/ "Enter the Pact Matrix. Or, how to decouple the release cycles of your microservices"), we need to also know whether a change to a service is compatible with its collaborators that are in the production environment.
