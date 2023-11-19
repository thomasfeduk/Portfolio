# Portfolio

- [API Endpoint Protection System](#api-endpoint-protection-system) (Demo)
- [AWS Sigv4 Implementation in Python, Javascript & Postman](#aws-sigv4-implementation-in-python-javascript--postman--gplv3) (GPLv3)
- [Kinesis-SLR](#kinesis-slr-gplv3) (GPLv3)
- [Data Models/DTO and data validation methodology](#data-modelsdto-and-data-validation-methodology)
- [Custom collections, abstract classes, @property and magic method overwriting](#custom-collections-abstract-classes-property-and-magic-method-overwriting)
- [Unit testing and testing methodology](#unit-testing-and-testing-methodology)

### API endpoint Protection System
A system my co-founder and I designed to restrict API endpoint communication to only approved company-issued clients via single-use generated ephemeral keys for each API call, with re-play attack protection.
 - https://fraudchallenge.thomasfeduk.com

### AWS Sigv4 Implementation in Python, Javascript & Postman  (GPLv3)
Below is an entirely-from-scratch AWS Sigv4 implementation I wrote first in Python, which I then ported to Javascript so I could execute AWS Lambdas, publish Kinesis messages and facilitate literally any other AWS service directly from Postman’s pre-flight scripts. Released under GPLv3

I tired of testing and trouble issues using multiple tools and wanted to do everything in Postman directly. I’ve also made extensive use of it in my discovery of an AWS infrastructure bug when I was designing tooling to dynamically spin up microservices with abstraction layers via Codestar and noticed some odd discrepancies and bugs in the official AWS SDK:

- https://github.com/thomasfeduk/aws_sigv4
- https://github.com/thomasfeduk/aws_sigv4/blob/main/python_example/aws_sigv4.py#L28

### Kinesis-SLR (GPLv3)
Below is a full project also released under GPLv3 which I call the Kinesis-SLR which is designed to assist with disaster recovery scenarios when Lambda failures occur and surgical and precise extraction and reply of Kinesis messages across all individual or all shards in a stream is needed:

- https://github.com/thomasfeduk/Kinesis-SLR

You can see the extensive permutations and complexities that must be accounted for and handled via the configuration options alone:
- https://github.com/thomasfeduk/Kinesis-SLR/blob/master/config-kinesis_scraper.yaml

### Data Models/DTO and data validation methodology
Utilizing DTOs/structured objects for data validation, sanitization and related mutations is a key aspect I adopt into nearly all of my work. I began working with dataclass and pydantic but a few years ago I ended up writing my own custom validation and DTO module I call [PropRules](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/common.py#L166) (short for Property Rules, soon to be released as a pip package) which drastically simplifies the amount of code necessary when specifying validation conditions.

- Example usage of PropRules class within my [Boto3GetRecordsResponse](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L270) and [GetRecordsIteration](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L31) classes.

### Custom collections, abstract classes, @property and magic method overwriting
An example of a custom collections class with magic method handling and associated restricted class:
- https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/common.py#L105

Usage of primary abstract class which has a shared base DTO for the [base AWS response model](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L18) and is extended upon for the [Input](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L82) and [Output](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L123) specific response:
- https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L18

### Unit testing and testing methodology
For mission critical systems especially, I make it a key point to ensure both positive and negative tests are used, as well as not only in method responses, but argument inputs, method call count and per-call examinations for multi-call situations (such testing iterators as they iterate through calls of a mocked method which returns different data for each subsequent execution).

Additionally, not only is code coverage important, but also that tests are written to cover the different logical branching patterns for the various conditional permutations. To cover many of these situations I make extensive use of generators fed into custom mocks as needed.

Below is an excellent example of a test for a multi-invocation call for a custom iterator ([RecordsCollection](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/kinesis_client.py#L245), [RestrictedCollection](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/common.py#L148), [Collection](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/includes/common.py#L122)) I wrote:
- [Seeding the generator](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1021)
- [Setting the expected state of each iteration](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1034)
- [Dynamically iterating through each expected iteration state within the _get_records() iterator mock](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1048)
  - [Asserting each state against the associated expected state](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1057)
- Separate from the per-iteration state checking above, [asserting exactly 3 iterations occurred](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1093) when [manually triggering the mocked iterator](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1082). This ensures app logic iterates exactly the expected number of occurrences given the input provided by the generator according to the [config file conditions](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1039) and that iteration stops short of the number of iterable results returned by the generator as we want it to [stop when 25 records are reached](https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_kinesis_client.py#L1040), regardless of the number of get_records() called they are spanned across.

It tests the following:
- Simulates records being fed from a series of Kinesis get_records() responses
- Distinctly checks each time the iterator is called
- That the iterator is called exactly a set number of times
- That each call has the expected inputs and outputs depending on the mock generator configuration

Regarding my PropRules DTO/validator module mentioned above: Because this is a module which provided tooling for developers, I came up with a method for building custom dynamic classes on a per-test-level which allows for testing classes with differing constructor conditions. It also demonstrates extensive positive/negative testing, exception handling and ensuring that not only the correct exception is raised but that it pertains to the expected argument/message:

 - https://github.com/thomasfeduk/Kinesis-SLR/blob/master/tests/test_common_proprules.py#L1015
