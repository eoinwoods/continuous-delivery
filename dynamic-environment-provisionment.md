# Dynamic Environment Provisioning in Securities Operations IT

## Context

Securities Operations IT utilises over a 100 software engineers split across 15 global locations.  The majority of effort spent by these teams is on two of the 7 core application components.  The software engineers are grouped into Feature teams, with each team working on its own prioritised piece of work with little or no common work across the teams; to all intents and purposes they work independently of each other.  Each team typically works on two features at a time.

We employ a [feature branch](http://martinfowler.com/bliki/FeatureBranch.html) model within the department with each feature segregated from the mainline until it is accepted (defined by the acceptance criteria for that feature) by our customers.

Our acceptance testing approach is in the middle of a transition towards a more optimal model.  While we continue to use Unit Testing approaches for the developer focused testing, we are replacing a home grown Acceptance Test Framework (ATF) with Open Source frameworks focused at a more appropriate level.

### Incumbent Legacy Testing Strategy
The ATF framework currently forms the core regression testing support for the department.  To allow this to run on developer machines and in a Continuous Integration model, a test version of the runtime caching framework has been developed.  Essentially this brings the distributed cache model into a single multi-threaded JVM.

For acceptance testing we largely employ a manual approach with developers executing the tests and users accepting the changes based on screenshots of the executed tests.  These screenshots are then further used as training for the end users.

There are a number of issues with the above testing approach:

1. The applications are deployed to a framework that is not the same as the one we use in production.  To allow it to run alongside other tests the component is deployed on a single JVM version of the runtime framework.  The single JVM version is materially different to the one we use in production.
1. The ATF tests are very hard to maintain with large amounts of duplication in the test context.  Additionally the ATF is a home grown testing framework where there are better products on the market that can be employed for less TCO.
1. The ATF framework is a black box testing framework and this approach is misused within the department: using black box testing when we should be testing the application components.
1. The manual testing is not repeatable and consumes a large amount of effort from the development teams.  Additionally the sign off process using screenshots for training is not maintainable as the applications evolve (they become out of date and irrelevant).
1. The captured 'evidence' is not relevant to any SOX/ICAP controls. 
1. The separation of testing and sign off means that often the wrong thing is built or the acceptance criteria is non-specific.


### Emergent Testing Strategy
There are two key changes to the current testing approach:

1. Perform the testing at the right level
1. Define feature acceptance criteria as tests

#### Test at the right level
The ATF tests currently operate at too high a level (at the system level), giving rise to lots of duplication across the tests.  By way of example, our 100+ swift formatting tests should work over only the formatting modules, and not the whole system.  We therefore see these 600+ ATF tests being broken out across two key dimensions:

1. The services and gateways.  In these tests, we will test the execution of the gateways and services against a mock cache.  They will be only focused on how these modules interact with objects in the cache and will assert based on the end state of objects in the cache.  This will mean that the individual services are tested at the appropriate level.  Alternative flow testing can happen in a more meaningful way (test data creation is more closely associated with the actual test scenarios).  These tests will be written in a structured syntax ([Gherkin](http://dannorth.net/whats-in-a-story/)) and the resultant documentation will form as the primary technical documentation for the component.
1. The Black Box tests.  In these tests, we will test the system end to end and therefore focus on the business processes that the application supports.  We will use a fully deployed instance of the application using the exact frameworks and technologies that exist in production today.  These tests will be written in a [natural domain language](http://www.agileinsider.org/2010/04/natural-language-automated-acceptance-testing/) and the resultant documentation will form the primary functional/business level documentation for the component.

In terms of the breakdown, we would expect the 600+ ATF tests to be converted into 40-50 end to end tests and perhaps 1000 module scoped tests.

#### Acceptance criteria as tests
The current model of: develop a requirement; test it; and capture screen shots for eventual acceptance by the users has some clear flaws (which have been realised in large monetary consequences, namely wasted team time).  In the new model, all tests would be written (set to ignored/draft) and committed prior to starting any feature.  These acceptance tests would be fully agreed with our users and will be detailed enough such that there is no confusion as to what we are developing and why.  In all cases these would form the materials used for sign off and for training of the application users.  These acceptance tests would then be 'curated', by which we mean optimised for duplication and scope, into the wider regression suite for the application.

These acceptance tests would be expected to be executed continuously from:

* Developer work stations
* Continuous integration tooling:
    * Feature branches
    * Mainline branches
* Smoke and user demonstration environments

### Test environments
We as a development organisation within the bank are continually being asked to drive down costs.  We would prefer to see this as becoming more efficient with the resources that we currently use and this is our internal strategy.  One of the biggest costs to us is in the physical resources that we use.  While we are provisioned virtual machines, they are a fixed specification and the lead time to change them is synonymous with physical hardware.  We currently employ the following fixed testing environments:

* Pre-production: a production like environment that is used for migration, deployment, operational, performance testing
* Production Patch: an environment dedicated to testing out of cycle changes (while other environments are busy)
* User Acceptance Testing: a functional testing environment connected to other departments for bank wide front to back testing
* Smoke: a developer owned environment for giving demonstrations to users and verifying the end to end state with manual tests

The above environment structure is really structured for an overall manual testing strategy with environment promotion of code versions.  If we were a single stream all working to common releases with fixed content and a waterfall development approach, this would work fine.

### Our environment requirements
We are not operating under a waterfall development model.  Are embrace the learnings from Lean and Agile and this allows us to be flexible in our working approach.  In order to develop software efficiently we have some basic asks as a development organisation:

1. Any developer should be able to release the component he is developing to an environment and test that it works as expected.
1. For any acceptance test that is automatically executed, it should run on an instance of the software exactly matching that of the intended deployment topology.
1. For any user demonstration (we expect one per feature every sprint) we expect this to be deployed on an instance exactly matching that of the production environment

This should apply for: the 100+ developers; the 500+ commit builds per sprint; the 30+ concurrently developed features; the 30+ demos per sprint.


## Why we need dynamic environments
__Testing on local machines__: the basic requirements for delivering a component build to an environment are such that developers are unable to deploy and run our application components to their local machines: Linux, minimum of 12GB plus 8 dedicated logical CPUs, plus a dedicated Oracle schema.  If you consider that the majority of our developers are on the offshore VDI machines with 1 logical CPU and 6GB available memory we are falling a long way short.  We cannot justify the cost of a dedicated environment for each team let alone for each developer.  Using a shared environment means contention between the 100+ developers to release and test their version of the code (this is the current model we have for Smoke and it simply does not work).  Additionally most developers need to deploy and test for a very short period of time and then they do not need the environment again for a few days.  We have for many years attempted to schedule environment allocation and this works to a point, but as you scale beyond a few teams this becomes an unconquerable problem.  The answer to the above is to allow for a dynamically provisioned environment.

__Ensure master is stable__: in the current testing approach, we need to converge the code branches to test them (single test environments).  This means that features are merged from their feature branches before we understand if they are integral and sound.  The effect of this is a constant stream of broken builds and a fundamental inability to release to production at any given point in time.  Additionally we spend a large amount of money (in people time) resolving issues and confusion around these breaks.  As we scale up this problem gets worse (as experienced with the introduction of the FXMM development stream).  We need a mechanism by which we can deploy and test the code from the branches and prove it is sound before we merge to the master branch.  To enable this we need to deploy the code from the branch and run the automated regression suite in full.  For this we either allow for contention of the static environments or we employ dynamic environment provisioning.  Our attempts to date have found that the former is impossible, and thus we really need a dynamic approach.

__Test the real application__: to enable the deployment of our current automated testing, we package our application against a mock (test only) version of our production caching framework that resides in a single JVM.  This fake environment breaches the principle of releasing the thing you have tested and therefore invalidates the testing.  To overcome this we need to be able to deploy our application components and test them in parallel to other deploy/test builds on other versions/branches of the codebase.

__Responding to change__: we currently operate under a model where we have a fixed number of test environments and each one leverages dedicated hardware to enable the environment.  If we have urgent requirements to support specific testing we are unable to deliver this at speed without compromising our existing environments (and by extension any projects using it).  New infrastructure provisioning (physical or virtual) is expected to take around 6 weeks to deploy.  We really need this on demand and often for only a few hours.  As such we need the ability to create new environments with fast turnaround that are dedicated to the testing group that needs it.

__Effective use of resources__: within the wider organisation we understand that virtualisation provides a better utilisation of hardware than physical provisioning.  The rationale behind this is that most non-production hardware is idle most of the time: time between test cycles; time as test teams are not in the office; time between developers deploying and testing in their own environments.  We believe that with the dynamic provisioning of environments comes the dynamic destruction of them when they are not used.  If we can bring the time to create and deploy to these environments down to below a minute we will view them as cheap disposable things with a short time to live.  This is how we achieve an optimal use of the resources to hand.

In summary we need dynamically provisioned environments because statically provisioned environments are not fit for purpose for our development approach.  The fact that the environments are static impedes our ability to move forward in an efficient manner and we believe will be cheaper for us in the longer term.


## Principles for Environment Management
1. Environmental __resource should be versioned__ alongside the software so that execution is repeatable
1. Environment Management __should not be costly__ and should be instant
1. Application __configuration should be minimised__
1. Test environments should __exactly match the production environment__, thus allowing testing of the production environment

### Rationale for the principles

1. Computer systems are a product of the hardware resources and the software that runs on them.  The importance of versioning software such that we can reliably rebuild a version with a minor modification is generally agreed upon.  The hardware provisioning is not as agreed upon.  Variance in the size and performance of the physical hardware between environments is dismissed as not affecting the functionality.  We often find this not to be the case.  Given the prevalence of environment configuration tooling and frameworks, such as [Puppet](http://puppetlabs.com/) and [Chef](http://www.getchef.com/) why are we still willing to make that compromise?  The principle here implies adoption of [Infrastructure as Code](http://jenssegers.be/blog/44/infrastructure-as-code) techniques and the embedding of infrastructure configuration into your deployment process
1. The management of an environment should be treated as a fully automated process.  There is little added value in a manual solution; we are looking for repeatability rather than creativity in the provisioning of environments.  The repeatability should be achieved through automation tooling such as Puppet or Chef.  The time to create a fully running environment should be near to immediate as it is really just a logical grouping of accesses to physical resources.
1. Software components allow for configuration so that environmental variances can be applied (here we distinguish between application wiring, application configuration and user driven static data).  The amount of configuration required will largely depend on the number and differences between the environments.  Through minimising the things that change in the application components we minimise the need for configurations in the system.  Largely these will be absorbed by the configuration of the dynamically provisioned environment, thereby the configuration management is separated from the concern of the application to the mechanism of deployment (following the encapsulation principles: [Single Responsibility Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle) or [Separation of Concerns](http://en.wikipedia.org/wiki/Separation_of_concerns) more generally.
1. One fundamental gap in most organisations testing strategy is "how to test production deployment before the release date".  In the normal process testing a system that is configured to be production is impossible (namespace violations at the least).  Through the use of a dynamically provisioned environment we can encapsulate the environment and test it as production.


## Proof of Concept done to date
Working closely with the CTO organisation we have conducted a Proof of Concept that aimed to show how we can use Virtualisation Technologies to dynamically provision an environment.  


### Execution details
The high level usecase is that everytime a change is made to the application codebase the following steps are automatically execusted:

1. Create and launch the virtual hardware:
    * A compute server to host our application
    * A Oracle database instance for the database
    * A test compute server to execute the tests, segregated from the deployed application
1. Configure the virtual hosts:
    * To resolve each others IPs (DNS)
    * To enable us to deploy and execute our applications
1. Create a pseudo-HUFS mount localy to release our application and any external dependencies
1. Release our application to the virtual hosts:
    * Configure and deploy our application
    * Import database from production dump and migrate it to the new version
    * Install a Apache AMQ messaging broker
1. Start our application
1. Execute the tests
1. Destroy the virtual machines, subject to successful test execution

This all takes less time than it took to write.

### Technology choices

The following technology choices were chosen for the simplicity of build plus the open availability within the bank and the wider open source community:

* __Openstack__: loosely binds us to a collection of virtualisation enabling technologies.  Essentially this opensource framework allows us to swap in an out different complimentary technologues so that we are not coupled to any single implementation (no lock in).  Additionally, Openstack provides a rich RESTful API that allows us to interact with it at the level we need.  This is likely to become the de-facto standard for virtualisation technologies in the years to come.  We note that right now it is painful to use, yet expect it to become less so in the future.
* __VMWare ESXi__: this is the hypervisor of choice to comply with our existing virtualisation infrastructure.
* __CentOS__: this is the closed Linux implementation to our internal standards.  Ideally we would have used an internal image.
* __Oracle__: this closely aligns us to our current production topology.

To abstract ourselves from any of these underlying technologies we have developed an in-house orchestration API that can be bound to any virtualisation technology (including container based solutions such as [Docker](http:..)).  Indeed it is felt by the team that the cloud technology space is still yound and developing such that any coupling to technologies could be costly in the longer term.


## What we need to take this forward

Cloud deployments operate at different levels as below:

1. __Infrastructure as a Service__ ([IaaS](http://en.wikipedia.org/wiki/Infrastructure_as_a_service)): 
1. __Platform as a Service__ ([Paas](http://en.wikipedia.org/wiki/Platform_as_a_service)):
1. __Software as a Service__ ([SaaS](http://en.wikipedia.org/wiki/Software_as_a_service)):


Platform as a service

We need the following two functions provided to us:
1. The ability to programatically create and destroy a working platform, in a similar manner to that of AWS or the Open Stack model
1. The ability to orchestrate 

In short, we are looking for a platform as a service layered model with the Infrastructure Services providing the [IaaS](http://en.wikipedia.org/wiki/Infrastructure_as_a_service) layering and the engineering teams providing the [PaaS] layer.






