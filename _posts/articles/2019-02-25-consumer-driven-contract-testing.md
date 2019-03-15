---
layout: post
title: "Consumer Driven Contract Testing"
excerpt: "Consumer driven contract testing is a method of verifying that services speak the same language. It is an alternative to traditional integration testing that gives you faster feedback "
modified: 2019-01-16 18:56:18 +0300
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

Consumer driven contract testing is a method of verifying that services (e.g. API consumer and an API provider) speak the same language.
By providing examples, API consumers set expectations on providers on how they should behave on specific inputs.
A set of expectations forms a contract that's produced by consumers and is shared with providers.
Contract obligations are verified by providers with tests that can be run in isolation, without having to set up integration testing environments.
That lets them evolve independently and get immediate feedback when they've broken any of their API consumers.
Contract testing can be used anywhere where you have two services that need to communicate with each other but becomes especially useful in environments with many services (e.g. microservice architecture).

Every story needs a hero.
Meet our heroes, [Alice and Bob](https://en.wikipedia.org/wiki/Alice_and_Bob), who are predominantly known for being the protagonists in many cryptography examples.
In our case, Alice and Bob got fed up of sending encrypted messages to each other.
They got in the field of software engineering and with all the hype around microservices, they've decided to ride the bandwagon.

Alice was responsible for creating the billing service.
Bob was tasked with building the customer service.
Billing service needs to access customer information to bill them.
To do that, it queries the `/customers/{id}` HTTP endpoint.

```
GET /customers/12

{
  "costumerId": 12,
  "name": "John Doe",
  ...
}
```

One evening Bob discovered that the API response payload has a typo.
Instead of returning [`costumerId`, it should be `customerId`](https://kathleenwcurry.wordpress.com/2015/09/08/easily-confused-words-costumer-vs-customer/ "Easily Confused Words: Costumer vs. Customer").
Being meticulous, Bob decided to immediately fix the humorous error and call it a day.

The next morning Bob was confronted by Alice.
"You broke the billing service!" she said.
Suddenly Bob realised, that he never tested the output of customer service with a running instance of billing service.

## Testing

Why didn't Customer service's tests catch this error?
Surely, Bob being a perfectionist, had created some tests.
Turns out he did.
Bob had created a test that verified the output of the `/customers/{id}` endpoint.
It failed after Bob's change but was fixed immediately.
Essentially, the test only assumed how the client of Customer service acts and does not reflect what the Billing service actually needs.
These kind of tests are cheap to maintain and fast to run, on the other hand, they're not trustworthy because if the API changes, it's not immediately clear, whether the clients still continue to work.

Having only two services - customer and billing - is a fairly trivial example.
Bob should have known, that Billing service uses Customer service.
But in a more large scale environment, with hundreds of services, it's not immediately clear, who are the clients of a specific API.
We'll look into this later.

## Integration testing

Testing two running instances of a service together seems like an integration problem that could be solved by integration tests.
Why didn't Alice and Bob have anything like that set up?
Since integration tests can have a different meaning in different contexts, here I refer to testing two services as together as an integration test (integrating two components of many).
Spinning up a Customer and a Billing service and exercising the communication between them should have caught the integration error.

Alice and Bob didn't have anything like that set up.

Integration tests can take a long time to run and fail for unrelated reasons (flaky network connection).
Therefore, in some cases, it might take multiple runs of the test suite to pass.
Alice and Bob decided that these kind of tests are not trustworthy and testing integrations this way is too expensive, not worth investing in yet.

This is even more difficult when the number of services increases.
There are different permutations of services that should be tested, services may have downstream services that should be mocked.

## End to end Testing

By end-to-end testing, I mean setting up the entire environment, services, databases etc. and exercising the application through public APIs.
Not saying that these tests aren't valuable, but to test the API of two specific services in a larger system via public APIs is costly.
The same limitations that are present in integration testing are present in end-to-end testing and are probably even more problematic.
Although, end-to-end tests cover a big chunk of a system and give us much confidence that the system is in working order if they pass, there are some notable disadvantages.

Setting up the environment where to run end-to-end tests is costly.
When Bob makes a change to Billing service, should he run tests together with the production version of Customer service?
What if Alice has made changes to Customer service as well?
Should Bob create another environment where the latest Billing service is tested together with the latest Customer service?

Compared to unit tests, end-to-end tests run slowly.
Depending on the size of the system, the feedback cycle from a commit until we know we did not break any consumers of our API can be hours.
What's more, the more moving parts we have in our tests, the more brittle and flaky they are.
There's a higher the chance that they fail not because of broken functionality but rather because of some unrelated reason (e.g. network glitch, invalid test data in DB).

In his book, [Building Microservices](https://samnewman.io/books/building_microservices/), [Sam Newman](https://twitter.com/samnewman) had the following to say about flaky tests.

> Flaky tests are the enemy. When they fail, they don’t tell us much. We re-run our CI builds in the hope that they will pass again later, only to see check-ins pile up, and suddenly we find ourselves with a load of broken functionality.
>
> When we detect flaky tests, it is essential that we do our best to remove them. Otherwise, we start to lose faith in a test suite that “always fails like that.” A test suite with flaky tests can become a victim of what [Diane Vaughan](https://en.wikipedia.org/wiki/Diane_Vaughan) calls the *normalization of deviance*—the idea that over time we can become so accustomed to things being wrong that we start to accept them as being normal and not a problem.
>
> <footer><strong>Sam Newman</strong> &mdash; <a href="https://samnewman.io/books/building_microservices/">Building Microservices</a></footer>

## Manual Testing

What about manual testing?
With the increased number of services in a system, this becomes unpractical.

## Consumer Driven Contract Testing with Pact

Alice and Bob thought that it would be good if they could verify the API between Customer and Billing service with tests that are as cheap to maintain and fast to run as unit tests.
As luck would have it, they went to a conference and learned about consumer driven contract testing (CDCT) with Pact.
[They were inspired](https://blog.daftcode.pl/hype-driven-development-3469fc2e9b22 "Hype Driven Development") and immediately started to dig into it more to understand whether it could be applied to their situation.

Pact is a contract testing tool.
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
Although, sharing pacts via AWS S3 or VCS repositories gets the work done, you are generally only verifying that the head versions of your services play nice with each other.
To be able to [deploy services independently](https://www.rea-group.com/blog/enter-the-pact-matrix-or-how-to-decouple-the-release-cycles-of-your-microservices/ "Enter the Pact Matrix. Or, how to decouple the release cycles of your microservices"), we need to also know whether a change to a service is compatible with its collaborators in a production environment.


|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ?          |
| Billing (`prod`)   |        ?            |       ✅          |

Alice's head version billing service and Bob's head version of customer service have been verified to be compatible with each other.
The same can be said for the production version of these services because at some point in time, they were also the head versions that were verified together and then deployed to prod together.
If Alice decided to deploy the latest version of billing service into production, she may run into the risk of breaking the API between billing and customer services because the latest version of billing has never been verified against the production version of customer service.

Here's where Pact Broker really starts to shine.
In addition to being a tool to share pact files, it will keep track of your consumer and provider versions and pact verifications between them in what is called ["The Matrix"](https://github.com/pact-foundation/pact_broker/wiki/Overview#the-matrix).
Every time Alice's billing service changes, it can publish a pact to the broker and a new entry will be added to the matrix.
Customer service's build can fetch the latest pact to verify that it is compatible with the latest changes in Billing. But the build could also fetch the pact published by the `prod` version of Billing service to ensure that latest version of Customer service is backwards compatible with `prod` billing service.

|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ?          |
| Billing (`prod`)   |        ❌           |       ✅          |

In this example, we can see that the latest version of Customer service is not compatible with the production version of Billing service.
Essentially, it would be unwise to go ahead and deploy the latest customer service to production since it would introduce a breaking change.

Similarly, when we'd like to deploy the latest Billing service to prod, we're unsure whether we're introducing a breaking change.
We would need to first check-out the `prod` version of Customer service and verify it against the latest Billing service.

|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ✅          |
| Billing (`prod`)   |        ❌           |        ✅          |

After running the verification, we can see from the Pact Matrix that latest billing service is compatible with production version of customer service.
No breaking changes were introduced and we can feel free to deploy our changes in Billing service to production.

tags, can-i-deploy, version tracking, allowing to deploy services independently, see who uses who, webhooks

## Breaking changes and backwards compatibility

In a microservices environment, it is important to make sure changes to a single application don't break its dependants (otherwise lock-step releases).
Coming back to the example in the beginning of this post where Bob modified the response payload of Customer service is an example of breaking backwards compatibility.
Although Bob caught the issue with tests, these tests only reflected his assumptions on how the API is used.

When Customer service's build has access to the latest pact file, Bob can always verify whether backwards compatibility has been broken or not by running the build locally.
There's no need to set up an integration testing environment and orchestrate multiple services to spin up.

## Implementing backwards compatible changes



Trust and teams inside the same org, different orgs, sometimes does not make sense to do CDCT when you don't trust the other party.

## Pact nirvana, workflow
