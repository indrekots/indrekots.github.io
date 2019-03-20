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

## Story time

Every story needs a hero.
Meet our fictional heroes, [Alice and Bob](https://en.wikipedia.org/wiki/Alice_and_Bob), who are predominantly known for being the protagonists in many cryptography examples.
In our case, Alice and Bob got fed up of sending encrypted messages to each other.
After careful consideration, they became software engineers and with all the hype around microservices, they've decided to ride the bandwagon.

Alice was responsible for creating the Billing service.
Bob was tasked with building the Customer service.
Billing service needs to access customer information to bill them.
To do that, it queries the `/customers/{id}` HTTP endpoint.

```
> GET /customers/12

< HTTP/1.1 200 OK
< {
<   "costumerId": 12,
<   "name": "John Doe",
<   ...
< }
```

One evening Bob discovered that the API response payload had a typo.
Instead of returning [`costumerId`, it should be `customerId`](https://kathleenwcurry.wordpress.com/2015/09/08/easily-confused-words-costumer-vs-customer/ "Easily Confused Words: Costumer vs. Customer").
Being meticulous, Bob decided to immediately fix the humorous error and call it a day.

The next morning Bob was confronted by Alice.
"You broke the Billing service!" she said.
Suddenly Bob realised, that he never tested the output of customer service with a running instance of the Billing service.

## Testing

Why didn't Customer service's tests catch this error?
Turns out they did.
Bob had created a test that verified the response of the `/customers/{id}` endpoint.
It failed after Bob's change but was fixed immediately.

Essentially, the test verified how the API should be consumed but not how the Billing service consumes it.
These kind of tests are cheap to maintain and fast to run.
If they fail, they pinpoint the underlying cause extremely well.
On the other hand, they're not trustworthy when it comes to testing integrations.
If the API changes, it's not immediately clear, whether API clients still continue to work.

Conversely, the same is true on the Billing service side.
It uses a mock Customer service that responds with dummy data.
The problem with tests that mock dependencies is that they make assumptions on how the real counterpart behaves.
But there's nothing there to actually verify this assumption.
Once the dependency has changed, the assumption is not valid anymore.

Having only two services—Customer and Billing—is a fairly trivial example.
Bob should have known, that Billing service uses Customer service.
But in a larger system, perhaps with hundreds of services, it's not immediately clear who the consumers of an API are.

## Integration testing

Testing Billing and Customer services together seems like an integration problem.
Spinning up both services and exercising the communication between them should have caught the integration error.
Unfortunately Alice and Bob didn't have anything like that set up.

Integration testing was considered but in the end it was decided not to implement them because it was not clear how well they would scale if the number of services is increased.
Classical integration testing works well in environments with fewer components.
Modern microservice architectures, on the other hand, have many moving parts.

<figure class="align-center">
  <img src="{{ '/images/2019-02-25-consumer-driven-contract-testing/architecture.png' | absolute_url }}" alt="">
  <figcaption>Modern architecture has evolved to have more moving parts. Image by <a href="https://medium.com/@benorama/the-evolution-of-software-architecture-bd6ea674c477">Benoit Hediard</a></figcaption>
</figure>

Setting up two services in a controlled environment and running the tests was deemed too costly, especially when a single service has to be tested together with multiple collaborators.
The time it takes for a CI build to finish was also taken into account.
Integration tests are definitely slower that testing against mocked dependencies.

Questions were also raised about independent deployability.
If service A and service B are tested together, does it mean they have to be deployed together as well?
Do services have to be tested with older collaborator versions to ensure backwards compatibility?

## Testing pyramid

Now is probably a good time to look at the testing pyramid.

<figure class="align-center">
  <img src="{{ '/images/2019-02-25-consumer-driven-contract-testing/unit-testing-pyramid.jpg' | absolute_url }}" alt="">
  <figcaption><a href="https://manifesto.co.uk/unit-testing-best-practices-java/">Testing pyramid</a></figcaption>
</figure>

It states that you should have a good balance of tests—unit tests, integration tests, end-to-end tests.
More focus should be put on the base of the pyramid.
Unit tests are the cheapest ones to create and maintain, they're the most targeted and fastest to run.
Moving higher towards the top of the pyramid, the slower and more costly the tests become.

## End to end Testing

Setting up all services and exercising the application through public APIs could have caught the integration issue between Billing and Customer services.
Unfortunately, Alice and Bob referred to the testing pyramid and did not put much focus on these types of tests.

The same limitations that are present in integration testing are present in end-to-end testing and are probably even more problematic.
Although, a single end-to-end test covers a big chunk of the system and can give us much confidence that the system is in working order, there are some notable disadvantages.

Setting up the environment where to run end-to-end tests is costly.
Compared to integration testing, all other collaborators would have to be configured and instantiated as well.

Compared to other types of tests, these are the slowest.
Therefore developers would have to wait a significant amount of time to get feedback whether their changes broke anything.
Although our example deals with only two services, in bigger systems it could take hours to finally find out whether everything is okay.

When a change is made to the Billing service, should it be tested together with the latest version of Customer service?
If Alice and Bob want to deploy Billing service independently from Customer service, they would have to set up another environment where the latest Billing service is tested with the Customer service that's currently present in the production environment.
Increasing the number of services in play would make the situation even more complex and costly to operate.

What's more, the more moving parts we have in our tests, the more brittle and flaky they are.
There's a higher the chance that they fail not because of broken functionality but rather because of a network glitch for example.

In his book, [Building Microservices](https://samnewman.io/books/building_microservices/), [Sam Newman](https://twitter.com/samnewman) had the following to say about flaky tests.

> Flaky tests are the enemy. When they fail, they don’t tell us much. We re-run our CI builds in the hope that they will pass again later, only to see check-ins pile up, and suddenly we find ourselves with a load of broken functionality.
>
> When we detect flaky tests, it is essential that we do our best to remove them. Otherwise, we start to lose faith in a test suite that “always fails like that.” A test suite with flaky tests can become a victim of what [Diane Vaughan](https://en.wikipedia.org/wiki/Diane_Vaughan) calls the *normalization of deviance*—the idea that over time we can become so accustomed to things being wrong that we start to accept them as being normal and not a problem.
>
> <footer><strong>Sam Newman</strong> &mdash; <a href="https://samnewman.io/books/building_microservices/">Building Microservices</a></footer>

// normalization of deviance challenger

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
Although, sharing pacts via AWS S3 or VCS repositories gets the job done, you are generally only verifying that the head versions of your services play nice with each other.
To be able to [deploy services independently](https://www.rea-group.com/blog/enter-the-pact-matrix-or-how-to-decouple-the-release-cycles-of-your-microservices/ "Enter the Pact Matrix. Or, how to decouple the release cycles of your microservices"), we need to also know whether a change to a service is compatible with its collaborators that are in the production environment.

The following is an example state in the pact matrix.

|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ?          |
| Billing (`prod`)   |        ?            |       ✅          |

Alice's latest version of billing service and Bob's latest version of customer service have been verified to be compatible with each other.
The same can be said for the production version of these services because at some point in time, they were also the head versions that were verified together and then deployed to prod together.
If Alice decided to deploy the latest version of billing service into production, she may run into the risk of breaking the API between billing and customer services because the latest version of billing has never been verified against the production version of customer service.

Here's where Pact Broker really starts to shine.
In addition to being a tool to share pact files, it will keep track of your consumer and provider versions and pact verifications between them in what is called ["The Matrix"](https://github.com/pact-foundation/pact_broker/wiki/Overview#the-matrix).

In order for new entries to be added to the matrix, every time Alice's billing service changes, it should publish a pact to the broker.
Similarly, for every change to the Customer service, its CI build should fetch the latest pact and verify whether it is compatible with the latest changes in Billing.
Once the verification passes, the build could then fetch the pact published by the `prod` version of Billing service to ensure that the latest changes in the Customer service are [*backwards compatible*](https://en.wikipedia.org/wiki/Backward_compatibility "Backward compatibility") with `prod` billing service.
This gives you immediate feedback whether you've introduced a breaking change.

The following table illustrates the state where the latest version of Customer service is not compatible with the production version of Billing service.

|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ?          |
| Billing (`prod`)   |        ❌           |       ✅          |

Essentially, it would be unwise to go ahead and deploy the latest Customer service to production since it would break the contract between Billing and Customer services.

Similarly, when we'd like to deploy the latest Billing service to prod, we're unsure whether we're introducing a breaking change.
We would need to first check-out the `prod` version of Customer service and verify it against the latest Billing service.

|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ✅          |
| Billing (`prod`)   |        ❌           |        ✅          |

After running the verification, we can see from the Pact Matrix that latest billing service is compatible with production version of customer service.
No breaking changes were introduced and we can feel free to deploy our changes in Billing service to production.

Later we'll have a look at a workflow that ensures pacts are published and verified continuously.

version tracking, see who uses who

### How does pact know what is deployed to production?

In the previous examples the Pact Broker knew what version of Billing and Customer service was deployed to the production environment.
*How did it knew that?
What integrations have to be put in place for that to happen?*
The answer is quite simple.
Pact Broker comes with a set of command-line tools that you can use to alter its state.
The feature we're currently interested in is [tagging](https://github.com/pact-foundation/pact_broker/wiki/Using-tags "Using tag").
After a deploy to your production environment, you can *tag* the service and version with whatever you like.
In the previous example, `prod` was used as the name for the tag.

### Can I deploy to production?

We have the Pact Matrix that knows what versions of services play nice with each other.
We've also tagged the versions that have been deployed to our production environment.
Therefore we're able to make the decision whether it is okay to deploy a specific version of a service to production.
Pact Broker comes with a command-line tool called `can-i-deploy` that can help you automate this check.

Before a deploy, Alice could run the following command to ensure that version `bfa3b32` of Billing service can be deployed together with other services in the production environment.
Here, the abbreviated git hash is used as a version number [but that's not strictly required and you could use a different versioning scheme](https://github.com/pact-foundation/pact_broker/wiki/Pacticipant-version-numbers "Pacticipant version numbers").

```
$ pact-broker can-i-deploy --pacticipant billing --version bfa3b32 --to prod
```

If there's a successful verification result between the production version of Consumer service and the Billing service that's about to be deployed, the command exists successfully and the deploy script can continue with the deployment.

```
Computer says yes \o/

CONSUMER         | C.VERSION | PROVIDER         | P.VERSION | SUCCESS?
-----------------|-----------|------------------|-----------|---------
billing          | bfa3b32   | customer         | a8261f9   | true     

All verification results are published and successful
```

If for some reason, the same verification is unsuccessful, the command exists with a non-zero exit code.

```
Computer says no ¯\_(ツ)_/¯

CONSUMER         | C.VERSION | PROVIDER         | P.VERSION | SUCCESS?
-----------------|-----------|------------------|-----------|---------
billing          | bfa3b32   | customer         | a8261f9   | false   

One or more verifications have failed
```

### Dependencies between services

Since Pact Broker records all consumer and provider pairs, It knows the dependencies between all of your services.
From the provider's perspective, this is an extremely valuable information.
If a breaking change is made to an API, it is immediately clear which consumers and which endpoints were broken.
This allows the provider team to plan ahead and evolve their API by gradually introducing backwards compatible changes and letting API consumers catch up.
Old API clients can be phased out and replaced with newer ones.
Unused and deprecated endpoints can be safely deleted since no consumer has declared their interest in it.

<figure class="align-center">
  <img src="{{ '/images/2019-02-25-consumer-driven-contract-testing/network_diagram.png' | absolute_url }}" alt="">
  <figcaption><a href="https://github.com/pact-foundation/pact_broker#network-diagram">Example network diagram displayed by Pact Broker</a></figcaption>
</figure>

Dependency information can be used in other clever ways.
For example, lets say, you've decided to control communication on the network level with firewall rules.
With some effort, it should be possible to configure network policies based on the information that's present in Pact Broker.
The same information could be used to autogenerate architecture diagrams as well.

It must be said that it does not come for free.
Trust must exists between the teams that decide to use Pact and consumer driven contract testing.
If some parties are not on board, it might be difficult to reap all of the benefits Pact provides.

## Continuous pact verification workflow

To take full advantage of Pact, an automated workflow needs to be set up.
The following is how Alice and Bob decided to work.
You might have to modify some steps to meet your team/org requirements.

1. Billing service's CI build generates a new pact file and runs local tests against it.
If they pass, the pact is published to the broker.

2. Using [webhooks](https://github.com/pact-foundation/pact_broker/wiki/Webhooks "Webhooks"), Pact Broker is configured to trigger Customer service's CI build when there's a change detected in the pact published by Billing service.
If no changes were detected, the new version of the consumer is *pre-verfied* to work with the same providers as the previous version did.

3. Customer service's CI build is configured to verify the latest consumer pacts.
Once that's successful, it will also fetch and verify pacts that that were published by its production consumers.
Verification results are published to Pact Broker, incrementally filling the compatibility matrix.

4. Customer and Billing service deploy jobs are configured with the `can-i-deploy` tool to verify it is okay to proceed with the deployment.

5. After the deployment has finished, all deployment jobs make sure to tag the deployed version in the Pact Broker with `prod` tag.
This ensures that services can be deployed independently with the help of `can-i-deploy`.

Consumer driven contract testing can be used to specify requirements to providers.
Think of it as TDD for integration testing.
Consumer team can specify a contract and give it to the provider team.
The provider team can try to verify the contract and see it fail.
What follows is very similar to test driven development - red, green, refactor.

This can lead to a situation where Alice adds a new expectation to the pact produced by Billing service and checks in her changes.
Pact Broker should trigger a CI build for Customer service and it should fail because the new interaction is not implemented on the provider side.


public api and pact
Trust and teams inside the same org, different orgs, sometimes does not make sense to do CDCT when you don't trust the other party.

you need to have buy in from all parties

* classical integration testing does not scale well
* alternative to integration testing
* e2e tests too costly
* let's services verify their interactions with collaborators in isolation
* faster feedback
* backwards compatibility
* evolve APIs
