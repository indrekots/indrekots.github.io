---
layout: post
title: "Consumer Driven Contract Testing"
excerpt:
modified: 2019-01-16 18:56:18 +0300
categories: articles
tags: [pact, cdct, testing]
image:
comments: true
share: true
published: false
aging: false
---

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
