---
layout: post
title: "The Challenges of Testing Microservices"
excerpt: "Microservices provide you with several benefits. But there's no question that they introduce a new set of challenges, one of them being testing."
modified: 2019-04-28 18:56:18 +0300
categories: articles
tags: [microservices, testing, cdct]
image:
  path: /images/2019-04-20-challenges-of-testing-microservices/cover.jpg
  thumbnail: /images/2019-04-20-challenges-of-testing-microservices/cover_thumb.jpg
  caption: "Photo by [Louis Reed](https://unsplash.com/photos/pwcKF7L4-no)"
comments: true
share: true
published: true
aging: false
---

When looking at many of the leading software engineering conferences out there, you'll immediately see that *microservices* are a topic that's discussed everywhere.
Entire tracks are dedicated for it.
Tools and products are being built to manage them.
There's definitely [benefits to building your software using microservices](https://dzone.com/articles/benefits-amp-examples-of-microservices-architectur "Benefits of Microservices Architecture Implementation").
But there's no question that they [introduces a new set of challenges](https://dzone.com/articles/challenges-in-implementing-microservices), one of them being testing.

## Story time

To illustrate the difficulties in testing microservices, I'm going to tell you a fictional story.
Meet our heroes, [Alice and Bob](https://en.wikipedia.org/wiki/Alice_and_Bob), who are predominantly known for being the protagonists in many cryptography examples.
In our alternative universe, they got fed up of sending and receiving encrypted messages to each other.
After careful consideration, they became software engineers and with all the hype around microservices, they've decided to ride the bandwagon.

Alice was responsible for creating *the Billing service*.
Bob was tasked with building *the Customer service*.
Billing service needs to access customer information to bill them.
To do that, it queries the `/customers/{id}` HTTP endpoint.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/overview.png' | absolute_url }}" alt="">
  <figcaption>Billing services queries Customer service in order to access customer information</figcaption>
</figure>

One evening Bob discovered that the API response payload had a typo.
Instead of returning [`costumerId`, it should've been `customerId`](https://kathleenwcurry.wordpress.com/2015/09/08/easily-confused-words-costumer-vs-customer/ "Easily Confused Words: Costumer vs. Customer").
Being meticulous, Bob decided to immediately fix the humorous error and call it a day.

The next morning Bob was confronted by Alice.
"You broke the Billing service!" she said.
Suddenly Bob realised, that he never tested the output of customer service with a running instance of the Billing service.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/broken.png' | absolute_url }}" alt="">
  <figcaption>Billing service didn't like the change in the API</figcaption>
</figure>

## Testing

Why didn't Customer service's tests catch this error?
Turns out they did.
Bob had created tests that verified the response of the `/customers/{id}` endpoint.
They started to fail after the payload was changed but tests were fixed immediately.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/failed-test.png' | absolute_url }}" alt="">
  <figcaption>Tests started to fail but were immediately fixed</figcaption>
</figure>

Essentially, tests verified how the API should be consumed but not how the Billing service consumed it.
These kind of tests are cheap to maintain and fast to run.
If they fail, they pinpoint the underlying cause extremely well.
On the other hand, they're not trustworthy when it comes to testing integrations.
If the API changes, it's not immediately clear, whether API clients still continue to work.

Conversely, the same is true on the Billing service side.
It uses a mock Customer service that responds with dummy data.
The problem with tests that mock dependencies is that they make assumptions on how the real counterpart behaves.
Unfortunately there's nothing there to actually verify this assumption.
Once the dependency changes, the assumption is not valid anymore.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/testing-with-mocks.png' | absolute_url }}" alt="">
  <figcaption>Mocks aren't updated automatically, when the real counterpart is modified</figcaption>
</figure>

Having only two services—Customer and Billing—is a fairly trivial example.
Bob should've known, that Billing service uses Customer service.
But in a larger system, perhaps with hundreds of services, it might not be immediately clear who the consumers of an API are.

## Integration testing

Testing Billing and Customer services together seems like an integration problem, right?
Spinning up both services and exercising the communication between them should've caught the integration error.
Unfortunately Alice and Bob didn't have anything like that set up.

Integration testing was considered but in the end it was decided not to implement them.
It was not clear how well they would scale if the number of services is increased.
Classical integration testing works well in environments with fewer components.
Modern microservice architectures, on the other hand, have many moving parts.

<figure class="align-center">
  <img src="{{ '/images/2019-02-25-consumer-driven-contract-testing/architecture.png' | absolute_url }}" alt="">
  <figcaption>Modern architecture has evolved to have more moving parts. Image by <a href="https://medium.com/@benorama/the-evolution-of-software-architecture-bd6ea674c477">Benoit Hediard</a></figcaption>
</figure>

Setting up two services in a controlled environment and running the tests was deemed too costly, especially when a single service has to be tested together with multiple collaborators.
Turns out that *the Inventory service* also consumes the API that Customer service provides.
The time it takes for a CI build to finish was also taken into account.
Integration tests are definitely slower than testing against mocked dependencies.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/integration.png' | absolute_url }}" alt="">
  <figcaption>In addition to verifying the compatibility of Customer and Billing service, Inventory service has to be taken into account as well</figcaption>
</figure>

Questions were also raised about independent deployability.
If version X of Billing service and version Y of Customer service were tested together, does it mean they have to be deployed together as well?
Alice and Bob wanted to avoid lock-step releases and wanted to deploy their services independently of others.
Therefore they would have to test for backwards compatibility with all services that are already deployed to production, increasing the cost of integration testing even more.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/backwards-compatibility.png' | absolute_url }}" alt="">
  <figcaption>Independent release cycle requires to test for backwards compatibility between the collaborators that are already deployed to production</figcaption>
</figure>

## Testing pyramid

Now is probably a good time to look at the testing pyramid.

<figure class="align-center">
  <img src="{{ '/images/2019-02-25-consumer-driven-contract-testing/unit-testing-pyramid.jpg' | absolute_url }}" alt="">
  <figcaption><a href="https://manifesto.co.uk/unit-testing-best-practices-java/">Testing pyramid</a></figcaption>
</figure>

It states that you should have a good balance of tests—unit tests, integration tests, end-to-end tests.
More focus should be put on the base of the pyramid.
Unit tests are the cheapest to create and maintain, they're the most targeted and fastest to run.
Moving higher towards the top of the pyramid, the slower and more costly the tests become.

## End to end Testing

Setting up all services and exercising the application through public APIs could have caught the integration issue between Billing and Customer services.
Unfortunately, Alice and Bob referred to the testing pyramid and did not put much focus on these types of tests.

The same limitations that are present in integration testing are present in end-to-end testing and are probably even more problematic.
Although, a single end-to-end test covers a big chunk of the system and can give us much confidence that the system is in working order, there are some notable disadvantages.

Setting up the environment where to run end-to-end tests is costly.
Compared to integration testing, **all other collaborators** would have to be configured and instantiated as well.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/collaborators.png' | absolute_url }}" alt="">
  <figcaption>In addition to Billing and Customer service, the system has other services as well</figcaption>
</figure>

Compared to other types of tests, these are definitely the slowest.
Developers would have to wait a significant amount of time before they get feedback on the status of their changes.
Although Alice and Bob deal with only a handful of services, in bigger systems it could take hours to eventually find out whether everything is okay.

End-to-end testing also does not solve the issue of independent deployability.
If Bob wants to deploy the Customer service independently of others, he would have to set up another environment where the latest Billing service is tested with all other collaborators that are currently present in the production environment.
Increasing the number of services in play would make the situation even more complex and costly to operate.

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/prod-collaborators.png' | absolute_url }}" alt="">
  <figcaption>Testing for backwards compatibility with collaborators in production</figcaption>
</figure>

## Flaky tests

The more moving parts we have in our tests, the more brittle and flaky they are.
There's a higher chance that they will fail not because of broken functionality but rather because of a network glitch for example.

In his book, [Building Microservices](https://amzn.to/2Ej0ZGq), [Sam Newman](https://twitter.com/samnewman) had the following to say about flaky tests.

> Flaky tests are the enemy. When they fail, they don’t tell us much. We re-run our CI builds in the hope that they will pass again later, only to see check-ins pile up, and suddenly we find ourselves with a load of broken functionality.
>
> When we detect flaky tests, it is essential that we do our best to remove them. Otherwise, we start to lose faith in a test suite that “always fails like that.” A test suite with flaky tests can become a victim of what [Diane Vaughan](https://en.wikipedia.org/wiki/Diane_Vaughan) calls the *normalization of deviance*—the idea that over time we can become so accustomed to things being wrong that we start to accept them as being normal and not a problem.
>
> <footer><strong>Sam Newman</strong> &mdash; <a href="https://amzn.to/2Ej0ZGq">Building Microservices</a></footer>

[Diane Vaughan](https://en.wikipedia.org/wiki/Diane_Vaughan) is a sociologist and a professor at the Columbia University.
She coined the phrase ["normalization of deviance"](https://en.wikibooks.org/wiki/Professionalism/Diane_Vaughan_and_the_normalization_of_deviance "Diane Vaughan and the normalization of deviance") in her book [The Challenger Launch Decision](https://amzn.to/2Q8AwAi "The Challenger Launch Decision: Risky Technology, Culture, and Deviance at NASA"), where she analysed the processes inside NASA that eventually lead up to the [Space Shuttle Challenger disaster](https://en.wikipedia.org/wiki/Space_Shuttle_Challenger_disaster "Space Shuttle Challenger disaster"). Over time an unsafe practice grew into something that was considered normal since it did not cause an immediate catastrophe. And then BOOM!

<figure class="align-center">
  <img src="{{ '/images/2019-04-20-challenges-of-testing-microservices/challenger.jpg' | absolute_url }}" alt="">
  <figcaption>January 28, 1986, Space Shuttle Challenger</figcaption>
</figure>

## Manual Testing

What about manual testing?
If we only had a single client and a server, it might sound reasonable.
But if we're dealing with several services, this approach becomes unpractical very quickly.

## Solution?

Testing services in isolation with stubs is quick and cheap but not trustworthy.
A change in a service requires to update all of its stubs that are used by its collaborators.
Since this is a manual step, it can be forgotten or mistakes can creep in.

Integration testing sounds like a good solution.
Unfortunately that's not cheap to maintain and fast to run compared to *testing in isolation*.
In addition, to be able to deploy services independently, we would also have to test for backwards compatibility with services already deployed to production.
This can result in long build queues and slow feedback loops.
The same is true for end-to-end testing.

Luckily there's a technique called Consumer Driven Contract Testing (CDCT) that can fix it.
It's like testing services in isolation but it guarantees that your stubs will never get out of date.
Stay tuned for the next post to learn about CDCT.
