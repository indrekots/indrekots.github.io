---
layout: post
title: "Rock-solid Consumer Driven Contract Testing Workflow With Pact"
excerpt: "To make the most out of Pact, you should put some time into automation.
This post will cover the steps you need to take to create a rock-solid workflow that helps you to reap the benefits of consumer driven contract testing."
modified: 2019-06-27 18:56:18 +0300
categories: articles
tags: [pact, cdct, testing, CI/CD]
image:
  path: /images/2019-06-10-pact-workflow/cover.jpg
  thumbnail: /images/2019-06-10-pact-workflow/cover_thumb.jpg
  caption: "Photo by [Campaign Creators](https://unsplash.com/photos/--kQ4tBklJI)"
comments: true
share: true
published: true
aging: false
---

In the previous post we had a look at [Consumer driven contract testing with Pact]({{site.url}}/articles/consumer-driven-contract-testing/ "Consumer Driven Contract Testing with Pact").
Consumer driven contract testing is a method of verifying that services (e.g. API consumer and an API provider) speak the same language.
By providing examples, API consumers set expectations on providers on how they should behave on specific inputs.
A set of expectations forms a contract that's produced by consumers and is verified by providers in isolation.
Pact is a contract testing tool that helps us implement these tests.

To make the most out of Pact, we should put some time into automation.
This post will cover the steps you need to take to create a rock-solid workflow that helps you to reap the benefits of consumer driven contract testing.
It is advised that you know the basics of [consumer driven contract testing with Pact]({{site.url}}/articles/consumer-driven-contract-testing/ "Consumer Driven Contract Testing with Pact").

## Sharing pacts

[In an earlier post]({{site.url}}/articles/consumer-driven-contract-testing/ "Consumer Driven Contract Testing with Pact"), we saw that a consumer of an API is responsible for setting expectations and generating the pact file.
Once that's done, the provider should have access to it in order to verify its validity.
How can this be done?
Ideally, we should automate it somehow.
Otherwise, it's very easy to forget to do it.
A change in the contract might never be picked up by a provider, defeating the entire purpose of consumer driven contract testing.

Pact files can be [shared in multiple ways](https://docs.pact.io/getting_started/sharing_pacts#alternative-approaches).
If our consumer and provider lived in the same VCS repository, pact files could be stored there as well.
Once the consumer CI build has finished, it could commit the generated pact files into the same repository.
A similar strategy could work also when both projects have their dedicated repository.
Once the consumer CI build has generated the pact file, an extra step can be added to the build process that commits the files to the provider's repository.

Another approach is to use an [artifact repository](https://jfrog.com/knowledge-base/what-is-an-artifact-repository/ "What is an artifact repository?").
In addition to storing the consumer application as a build artifact, the CI build could also publish pacts as build artifacts.
On the provider side, you would need to figure out how to construct the URL that points to the latest pact file in your artifact repository to access it.

Additionally, a consumer CI build could store the generated pacts on a shared storage that's accessible to the provider build.
But if you want to take the most out of Pact, the recommended way to share pact files is to use [Pact Broker](https://github.com/pact-foundation/pact_broker).

## Pact Broker

The recommended way to share pacts is to use the [Pact Broker](https://github.com/pact-foundation/pact_broker).
After the consumer has successfully generated a pact file, it should publish it to the broker.

<figure class="align-center">
  <img src="{{ '/images/2019-06-10-pact-workflow/publish-pact.png' | absolute_url }}" alt="">
  <figcaption>Generated pact file is published to the Pact Broker</figcaption>
</figure>

Although, sharing pacts via network file sharing or VCS repositories gets the job done, you are generally only verifying that the head versions of your services play nicely with each other.
To be able to [deploy services independently](https://www.rea-group.com/blog/enter-the-pact-matrix-or-how-to-decouple-the-release-cycles-of-your-microservices/ "Enter the Pact Matrix. Or, how to decouple the release cycles of your microservices"), we need to also know whether a change to a service is compatible with its collaborators that are in our production environment.

<figure class="align-center">
  <img src="{{ '/images/2019-06-10-pact-workflow/recap.png' | absolute_url }}" alt="">
  <figcaption>Verifying whether the head versions of Billing and Customer service play nicely with each other</figcaption>
</figure>

If the latest version of Billing service generates a pact file and the latest version of Customer service verifies it, we know that we can safely deploy these two services to production together.
But if we would like to deploy only the head version of Billing service independently, we may run into the risk of breaking the API between billing and customer services because the latest version of billing has never been verified against the production version of customer service.

## Pact Matrix

Here's where the Pact Broker really starts to shine.
In addition to being a tool to share pact files, it will keep track of your [consumer/provider versions and pact verifications](https://docs.pact.io/getting_started/versioning_in_the_pact_broker "Versioning in the Pact Broker") between them in what is called ["The Matrix"](https://github.com/pact-foundation/pact_broker/wiki/Overview#the-matrix).

The following is an example state in the Pact Matrix.

|                    | Customer (`latest`) | Customer (`prod`) |
|--------------------|---------------------|-------------------|
| Billing (`latest`) |        ✅           |        ?          |
| Billing (`prod`)   |        ?            |       ✅          |

Here we can see that the latest versions of Billing and Customer service are compatible with each other.
Additionally, there's a successful pact verification between the production versions as well.

In order for new entries to be added to the matrix, every time the Billing service changes, it should publish a new pact to the broker.
Similarly, for every change to the Customer service, its CI build should fetch the latest pact and verify whether it is compatible with the latest changes in Billing.
Once the verification passes, Customer service's CI build should then fetch the pact published by the `prod` version of Billing service to ensure that the latest changes in the Customer service are [*backwards compatible*](https://en.wikipedia.org/wiki/Backward_compatibility "Backward compatibility") with `prod` billing service.
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

After running the verification, we can see from the Pact Matrix that latest Billing service is compatible with production version of Customer service.
No breaking changes were introduced and we can feel free to deploy our changes in Billing service to production.

Later we'll have a look at a workflow that ensures pacts are published and verified continuously.

### How does pact know what is deployed to production?

In the previous examples the Pact Broker knew what version of Billing and Customer service was deployed to the production environment.
*How did it know that?
What integrations have to be put in place for that to happen?*
The answer is quite simple.
Pact Broker comes with a set of command-line tools that you can use to alter its state.
The feature we're currently interested in is [tagging](https://github.com/pact-foundation/pact_broker/wiki/Using-tags "Using tag").
After a successful deploy to a production environment, we can *tag* the service and version with whatever we like.
In the previous example, `prod` was used as the name for the tag.

### Can I deploy to production?

We have the Pact Matrix that knows what versions of services play nicely with each other.
We've also tagged the versions that have been deployed to our production environment.
Therefore we're able to make the decision whether it is okay to deploy a specific version of a service to production.
Pact Broker comes with a command-line tool called `can-i-deploy` that can help us automate this check.

Before deploying the Billing service, we should run the following command to ensure that version `bfa3b32` of Billing service can be deployed together with other services in the production environment.
Here I've used the abbreviated git commit hash as the version number [but that's not strictly required and you could use a different versioning scheme](https://github.com/pact-foundation/pact_broker/wiki/Pacticipant-version-numbers "Pacticipant version numbers").

```
$ pact-broker can-i-deploy --pacticipant billing --version bfa3b32 --to prod
```

If there's a successful verification between the production version of Customer service and the Billing service that's about to be deployed, the command exits successfully and the deploy script can continue with the deployment.

```
Computer says yes \o/

CONSUMER         | C.VERSION | PROVIDER         | P.VERSION | SUCCESS?
-----------------|-----------|------------------|-----------|---------
billing          | bfa3b32   | customer         | a8261f9   | true     

All verification results are published and successful
```

If for some reason, the verification was unsuccessful, the command exits with a non-zero exit code.

```
Computer says no ¯\_(ツ)_/¯

CONSUMER         | C.VERSION | PROVIDER         | P.VERSION | SUCCESS?
-----------------|-----------|------------------|-----------|---------
billing          | bfa3b32   | customer         | a8261f9   | false   

One or more verifications have failed
```

### Dependencies between services

Since Pact Broker records all consumer and provider pairs, it knows the dependencies between all of the services.
From the provider's perspective, this is an extremely valuable information.
If a breaking change is made to an API, it is immediately clear which consumers and which endpoints were broken.
This allows the provider team to plan ahead and evolve their API by gradually introducing backwards compatible changes and letting API consumers catch up.
Old API clients can be phased out and replaced with newer ones.
Unused and deprecated endpoints can be safely deleted since no consumer has declared their interest in them.

<figure class="align-center">
  <img src="{{ '/images/2019-02-25-consumer-driven-contract-testing/network_diagram.png' | absolute_url }}" alt="">
  <figcaption><a href="https://github.com/pact-foundation/pact_broker#network-diagram">Example network diagram displayed by Pact Broker</a></figcaption>
</figure>

Dependency information can be used in other clever ways as well.
For example, let's say, you've decided to control communication on the network level with firewall rules.
With some effort, it should be possible to configure network policies based on the information that's present in Pact Broker.
The same information could also be used to autogenerate architecture diagrams.

It must be said that it does not come for free.
Trust must exists between the teams that decide to use Pact and consumer driven contract testing.
If some parties are not on board, it might be difficult to reap all of the benefits that Pact provides.

## Continuous pact verification workflow

So far we've seen that there are several steps that you must follow in order to take full advantage of Pact.
Therefore it would be wise to set up an automated workflow that makes sure you won't miss any of them.
The following is a workflow that makes use of all of the Pact features we've covered so far.
You might have to modify some steps to meet your team/organisation requirements.

* Billing service's CI build generates a new pact file and runs local tests against it.
If they pass, the pact is published to the broker.

<figure class="align-center">
  <img src="{{ '/images/2019-06-10-pact-workflow/billing-ci.png' | absolute_url }}" alt="">
  <figcaption>Billing service's CI build</figcaption>
</figure>

* Using [webhooks](https://github.com/pact-foundation/pact_broker/wiki/Webhooks "Webhooks"), Pact Broker is configured to trigger Customer service's CI build when there's a change detected in the pact published by Billing service.
If no changes were detected, the new version of the consumer is *pre-verfied* to work with the same providers as the previous version did.

* Customer service's CI build is configured to verify the latest consumer pacts.
Once that's successful, it will also fetch and verify pacts that that were published by its production consumers.
Verification results are published to Pact Broker, incrementally filling the compatibility matrix.

<figure class="align-center">
  <img src="{{ '/images/2019-06-10-pact-workflow/customer-ci.png' | absolute_url }}" alt="">
  <figcaption>Customer service's CI build</figcaption>
</figure>

* Customer and Billing service deploy scripts are configured with the `can-i-deploy` tool to verify whether it is okay to proceed with the deployment.

* After the deployment has finished successfully, all deployment scripts have to tag the deployed version in the Pact Broker with `prod` tag.
This ensures that services can be deployed independently with the help of `can-i-deploy`.

## Summary

Pact is a powerful tool that helps you implement consumer driven contract testing.
In order to make the most out of Pact, it is strongly advised that you start using Pact Broker as well.
Putting in some effort to implement the steps presented in this post and integrating Pact Broker with your CI/CD pipeline will greatly help you in deploying your services independently and avoiding breaking changes.
