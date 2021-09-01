General:
--------

This version has updated **in September 2020** and is **accepted** by all **DeepSea teams**. Additionally, the payment organization live this manifest as well. Discussions with other departments are ongoing, while most of it is already accepted company wide.

These values & principles are widely accepted and can be seen as mandatory. In rare specific cases, deviations may be neccessary, but should be avoided. If unsure, discuss these cases with TDs of affected teams or within the TD-Round/Tech-Circle

Values
------

Describe, what is important to us and where we want to put an emphasis. They should guide and influence our daily work and decisions and describe a vision. All values are expected to be used with common sense.

### Tech follows Business

The whole goal of our effort is to create a blazing business value. All technical decisions should support the business. The sooner the better. Business sometimes require fast changes of a team's roadmap. We want to support that. The whole Team also has the responsibility to develop business ideas, not only Business focusing team members. Teams build products for our partners & customers, not just software artifacts. Business value can only be gained by delivering software to production. Documents or concepts have no inheritent value.

Additionally we value the benefit for the company higher than the benefit for our team or ourselves.

### Move fast, fail fast

We value to quickly iterate in short build → measure → learn cycle cycles. Failing fast is a strongly recommended approach, often realized with fully automated pipelines, to see mistakes as soon as possible in the development process.

We believe it is important that teams contain the necessary people and power to build features and products with as few dependencies as possible to other teams. We consider Conway's law and split our domains in a way that allows for loose coupling and increased team autonomy.

Uncertain business requirements become clearer and tangible when teams start to build and deploy potential shippable software products. Having a running software with minimal functions is more valuable than discussing about concepts.

We appreciate refactoring and follow pareto principles to fulfill business requirements. Iterative design and feedback loops with partners & customers are helpful and tend to a customer focused solutions. Software is designed to fail, outages are not fully preventable. We focus on the meantime to recover instead of optimizing meantime between failures.  

### Low technical barriers

Although we thrive for independant teams, the teams are not alone. We help each other and reduce technical barriers and increase knowledge exchange, we value the standardization of technologies and processes across the teams, where beneficial. We prefer open standards above own standards.

### Coverage of non functional aspects 

We put a special focus on non-functional aspects of our software. We consider the criticality and maturity of a product when deciding the level of effort required for resilience. We react with urgency if problems are detected and we apply our security best practices to all our applications. We measure performance of our software even in heavy high load scenarios and assure the appropriate responsetimes. Infrastructure setup is highly automated to assure we can rebuild the whole environment in case of disaster recovery. Quality is a non discussable dimension and achieved by following the test pyramid.

### Full ownership and clear responsibility

We believe teams are responsible for supporting, monitoring, alerting, operating and retiring including quality aspects of everything they build and maintain - You build it, you run it! Responsibilities and ownership should always be clear and published. Everything that is used in production or shared across multiple teams must be owned. We strive for "Aligned autonomous cross functional teams", which means a team should be able to fulfill its assignment as autonomously as possible, but should also align on concepts and technical solutions with the other teams, where synergies could be benefical.

The teams are also responsible for their costs and strive for optimization.

Principles
----------

To provide more details and rationale on how to apply these Values, we derive and define a set of the most important principles. These principles are roughly categorised into topics:

-   Architecture
-   Operational
-   Technology & Practices

### Architecture

#### Bounded Contexts & Vertical Systems

IT Systems should be build for the business needs, so we align our IT products with the business domains. Therefore we are using bounded contexts and domain models which reflect the business language for the design of our IT products.

**Rationale**

Full ownership and clear responsibility is one of the most important aspects, if teams should be able to act independently from each other. Therefore we try to identify bounded contexts followed by the Domain Driven Design approach. Ideally nothing should be shared between the identified contexts, especially not the domain model, which leads to IT product which can be assigned to one bounded context and can be owned by one team. The IT product includes everything to fullfill the business need, including data, business logic and UI fragments.

**Implications**

-   Every identified context has its own domain model, including owning its own data, the business logic & UI
-   Communication between contexts is designed by using domain events
-   Use the domain language (ubiquitous language) in your software artifacts. Avoid using anemic domain models
-   For UI composition we prefer UI composition using micro-frontend patterns over a frontend/UI monolith

**Points for Discussion**

-   Is the boundary of your context clear?
-   Can you name your boundary events?
-   How is the relationship to your Up- and Downstream contexts?
-   Do you own your context fully?
-   Do you have a clear understanding of your domain model  and is it shared with the business side to build up a ubiquitous language?

**Motivating values**

-   Full ownership and clear responsibility
-   Tech follows Business

#### Non Blocking Communication

We prefer non blocking technical communication between systems to achieve technical autonomy from other systems.

**Rationale**

Blocking calls to remote systems can lead to long running threads waiting for outside dependencies. This can completely paralyze a system, as more and more threads move into that state until the system can no longer react to new requests. If multiple services are chained, aside of the increased latency, can lead to cascading errors, affecting all involved system. Recovery might also be affected as for example restarted services might immediately get a lot of traffic, bringing them immediately down again.

Technical communication should follow a standard non blocking pattern such as pubsub, feeds or webhooks.

**Implications**

-   We reduce the impact of remote failures by communicating non blocking via events, where possible. Microservices publish streams of events, which interested services consume asynchronously.
-   Communication with other systems becomes non blocking: even when some functionality is affected temporarily, the system continues working.
-   Where necessary, events should contain additional ordering information
-   Implement your services as an [idempotent receiver](https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html) where possible
-   If a synchronous call is necessary, we make use of circuit breakers, timeouts and bulkheads (see service degradation)

**Questions to be considered**

-   What is the right way to model events? Event carried state transfer (and thus include the entire state in an event), event notification, command messages, or only deltas published?
-   Are you just publishing your transaction log as events or are your events tailored to actual business events and the respective consumers? Avoid [Puncturing encapsulation with change data capture](https://www.thoughtworks.com/radar/techniques/puncturing-encapsulation-with-change-data-capture) between business contexts.
-   Are the events idempotent? Is there are need to make ordering an explicit property of the events?
-   Does the event inform about a state change or is it actually a command (customer email updated vs. publish refund notification).
-   Is the event catering multiple consumers and is generic (changes are harder) or is it tailored for a specific consumer (change affects only one system)
-   Do the events contain GDPR/PII information? Are all consumers allowed to have access to them?

**Motivating values**

-   Coverage of non functional aspects
-   Full ownership and clear responsibility

#### Small and Simple (Microservices)

Build services small and simple. Design them around business capabilities, based on a domain model mostly with state and behaviour.

**Rationale**

A service should be big enough to offer a valid business capability, but small enough to be handled by [two pizza sized](https://www.borisgloger.com/blog/2019/07/17/wie-microservices-und-die-two-pizza-rule-das-entwicklungsteam-optimal-unterstuetzen/)team of. In practice, a team may be able to own and run a large number of small services, or a smaller number of larger services.

Building small and simple makes systems easier to understand. But with more systems the interactions between the system can get more complex. Therefore it is not the goal to design as much little microservices as possible, but to design a system landscape within a teams domain, where the different subdomains are implemented as independent as possible. Subdomain overarching microservices should be avoided.

**Implications**

-   Prefer microservices over monoliths
-   Ensure sure we leverage the benefits and make them independently deployable, avoiding a distributed monolith
-   Model your Microservices based on your domain (a domain can have multiple Microservices)
-   Consider using Anti-Corruption-Layers to decouple your services from the surrounding domains
-   Your Microservices should be independently testable
-   Take [12 factor app](https://12factor.net/) into consideration when designing your microservices

**Questions to be considered**

-   Are your services independently deployable and testable?
-   Are you building a distributed monolith?
-   Are your microservices sliced based on your domain or is the slicing of a rather technical nature?
-   Are you building micro- or nano-services?
-   Does your Microservices cater to multiple domains?
-   Are your Microservices actually independent or are they integrating on the database layer?
-   [Are you tall enough to use Microservices](https://martinfowler.com/bliki/MicroservicePrerequisites.html)?

**Motivating Values**

-   Full ownership and clear responsibility
-   Move fast, fail fast
-   Low technical barriers

#### Evolutionary Architecture

Systems and architectures should be designed and built to evolve independently from other systems.

**Rationale**

At this point in time we know less than what we'll know in 6 months. Our software should be able to evolve as we learn more. It should evolve both at a code level and also at a service level within the business responsibilities.

**Implications**

-   Defer accidental complexity as much as possible. We focus on addressing current needs, accept that we are poor at predicting future needs.
-   Systems and processes will need to be exposed through clearly defined APIs so that the underlying implementation can change and evolve without impacting the wider system. (Note: this does not mean that a component or system's *internal* APIs necessarily need to be approached in the same way.)
-   Consider patterns such as [TolerantReader](https://martinfowler.com/bliki/TolerantReader.html), backwards compatibility and the [Principle of Robustness](https://en.wikipedia.org/wiki/Robustness_principle), if applicable or useful
-   Standards that are derived from working software and not just working groups are preferred.
-   Sometimes gradual/incremental change is insufficient as requirements change, so consider a [Sacrificial Architecture](https://martinfowler.com/bliki/SacrificialArchitecture.html) as a pattern. Don't be afraid to throw away and re-write, as this is often faster and will lead to better quality. If we have kept things "Small and Simple" this should not turn into a painful [Big Rewrite](http://chadfowler.com/2006/12/27/the-big-rewrite.html).
-   Consider the use of Standards (software and protocols). If they reasonably do not fit your needs, evaluate software or protocols which are highly adopted. Industry standards over company standards to cover the needs of non-company clients.

**Questions to be considered**

-   Are you implementing actual or anticipated requirements?
-   Did you design using a BDUF (Big Design Up Front) approach?
-   Do your service interfaces and protocols rely on well known standards ore are they tightly coupled to a proprietary 3rd party tool/protocol?
-   Does your software rely on well known standards or are they tightly coupled to a proprietary 3rd party tool/protocol?

**Motivating Values**

-   Move Fast, Fail Fast
-   Low technical barriers

### Operational

#### Cloud Native

We prefer services directly provided by AWS or GCP. Teams have the ability to choose, which cloud provider fits best to the needs, but should always be prepared to explain why a specific cloud provider was selected.

**Rationale**

We want fully leverage the cloud and use services provided by AWS/GCP wherever possible. This limits the technical landscape and also removes the burden of operating and scaling backing services. Avoid multicloud setups in single teams but follow the overall company strategy to achieve a mixture ofdifferent providers.

A team-specific vendor-lockin is accepted by the means, that teams are using high-level services like databases or serverless architectures, which are not equally existing on other cloud providers. Still the teams should follow good deveopment principles like decoupling business logic from infrastructure components.

**Implications**

The overall priority for selecting infrastructure and supporting services:

1.  Prefer cloud services provided by primary cloud provider in team
2.  If there is nothing, use offers operated by a third party as Software-as-a-Service Service 
3.  Only if there is nothing that fulfils our requirements, build or operate your own service within your cloud account in the following priority
    1.  self hosting an active open source solution
    2.  self hosting 3rd party paid solution
    3.  self implementing and hosting a solution

**Questions to be considered**

-   Which cloud provider offers the best technical fit for our software?
-   How have other teams solved that particular problem?
-   Is the chosen technology really necessary?
-   Is a support level appropriate/needed for a selected component? 

**Motivating Values**

-   Move fast, fail fast
-   Low technical barriers

#### Scale Horizontally

System components should be horizontally scalable where possible.

**Rationale**

It is cheaper to scale services by adding inexpensive resources like commodity servers or database nodes rather than buying larger and larger pieces of hardware which at a given point are unable to scale further. By embracing horizontal scalability early and designing our systems to work in this manner we eliminate the cost of changing at a later date.

**Implications**

-   State data cannnot be saved in the applications memory. Consider the use of shared databases or in-memory data grids like memcached or redis)
-   Services should strive to be idempotent

**Questions to be considered**

-   Is ACID functionality required or is eventual consistency (BASE) sufficient?

**Motivating Values**

-   Move Fast, Fail Fast

### Technology & Practices

#### Continuous Delivery and Deployment

Strive for short release cycles and a limited number of changes; automating the delivery pipeline makes this possible

**Rationale**

Small releases tend to have fewer bugs. Continuous Integration usually refers to integrating, building, and testing code within the development environment. Continuous Delivery and Deployment builds on this, dealing with the final stages required for production deployment. This is done to reduce deployment risk, show believable progress, and reduce the time to get feedback from real users.

**Implications**

-   Use feature toggles, not only to separate deployment from release and let business decide when to turn on a feature, but also as a mechanism to quickly turn off broken features and limit their impact. Also remind different feature toggle sets on different stages/environments.
-   Use canary testing for your new deployments to identify problems early.
-   Have an automated test suite with high coverage to support rapid deployments
-   Use Consumer Driven Contract Tests to ensure changes are not breaking dependent systems
-   Consider push-button deployments of any version of the software to any environment on demand, especially for rapid software rollbacks.
-   We use a single trunk based CI build as the beginning of our pipline

**Questions to be considered**

-   What manual gates are in place? Is it useful to have one?
-   Can we release straight to production?
-   Is a green build trustable for all team members? if not: What would be the steps to achieve this?
-   Which trust building actions like tests, CDCs and so on need to be implemented to achieve sufficient trust.

**Motivating values**

-   Low technical barriers
-   Move fast, fail fast

#### Sensible Defaults

We minimize technology variation and instead leverage sensible defaults across teams. 

**Rationale**

We deliberately minimize the variety of technologies in our system to drive efficiency, and, at the same time, have a managed program of innovation. To achieve this, we monitor actual technologies in use, gravitate to those most appropriately addressing the given challenges and selectively experimenting with new technologies to find those that will improve our products. We maintain a technology radar to have a good overview about the current technology selection and usage. Besides the radar we maintain a set of skeleton templates to quickstart newly formed teams.

The radar is a guideline recommendation, usage of not listed technologies (frameworks, languages, tools) is not prohibited, but teams need to reason why they have chosen a specific technology.

**Implications**

-   Use the technology radar for technology selections
-   Discuss the need of new technologies with others
-   Regulary actively contribute and challenge radar

**Questions to be considered**

-   How often should the technology radar be updated?
-   What are the criterias for the rings (trial, access, adopt & hold) for the tech radar?

**Motivating values**

-   Low technical barriers
-   Tech follows Business

#### Use Low-Tech Coupling (aka Smarts in the Nodes not the Network)

We prefer simple, proven technologies for communication

**Rationale**

Low-tech coupling reduces issues resulting from changes in communicating systems, and can reduce complexity and dependencies. The reduced complexity makes it easier to locate and identify problems. Furthermore it makes it easier for new teams to participate and for example tap into event streams. Proven, simple solutions reduce the mental load when talking about service to service communication.

**Implications**

-   Use DNS for service discovery
-   Use interoperable protocols like HTTP for synchronous communication instead of, for example, RMI
-   Prefer human readable formats like JSON for payloads (instead of protobuf or msgpack)
-   Prefer event streaming from the data master directly instead of using integration patterns like Enterprise Service Bus.

**Questions to be considered**

-   What kind of problems would a more complex solution fix? Do we actually have that problem?
-   Do we observe challenges because of an in use low tech solution across the DeepSea?
-   Is the performance gained by more complex solutions necessary in the particular context it should be used?

**Motivating Values**

-   Low technical barriers
-   Coverage of non-functional aspects

#### Make Decisions in Public

We prefer to make cross-team decisions in public, ask for feedback and document the conclusion

**Rationale**

A lot of technical and design decisions affect more than one team or even the entire DeepSea. We would like to leverage our combined knowledge and make decisions in public, allowing everyone interested to participate to come to the best possible conclusion. While this does not mean that decisions will need to be majority votes but that everybody is given the chance to provide feedback and arguments.

Often some time after a decision has been made, especially if it causes unanticipated challenges, it helps a lot to understand the context, knowledge and anticipated consequences at the point in time the decision has been made.

**Implications**

-   We value RFCs (Request for Comment) approach on a cross-team level where possible. The RFC should contain the challenge, possible solutions and at some point the decision made and why it has been made.
-   We keep the RFCs at a central place for easy lookup
-   Within a team, we recommend to leverage [ADRs (Architecture Decision Records)](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions) to document major architectural or technical decisions

**Questions to be considered**

-   Has there been enough time to actually provide comments to an RFC?
-   Is the decision properly documented at an easy to find location?
-   Has your team documented important decisions in the form of an ADR?

**Motivating Values**

-   Low technical barriers
-   Tech follows Business

#### Security First 

Systems will be designed and maintained with the assumption that they *will* be attacked and possibly compromised by **anybody**.

**Rationale**

The cost of handling a security breach^[1](http://engineering-principles.onejl.uk/#fn_1)^ is significantly greater than the cost of hardening the system in the first place. If a breach does occur, we can minimise the scale and cost by detecting it as soon as possible, and having a well-planned response enables a fast time to close the breach and assess its impact.

By focussing on security from the beginning we mitigate not only the financial impact but also the reputational impact that would be incurred in the event of a breach.

**Implications**

-   Stick to our [Security Dojo](https://dojo.sec.otto.de/#/) (Security Application checklist) and partitipcate security guild (opSec-Security). 
-   Have a security champion/catalyst and consider the security aspects of every story
-   Always use TLS and make sure the caller of your service is authenticated and authorized. That means "HTTPS everywhere, not HTTP."
-   Use threat modelling workshops (OTTO provides moderated team workshops with security experts) to identify threats that are inherent to the domain (e.g. amplification attacks where a small number of  malicious database query causes a service outage)
-   Design your services and implementations in a secure by default manner (e.g. a service by default enables TLS and authorization/authentication instead of explicitly activating it)
-   Have monitoring in place to automatically see brute force attacks or unusual requests or unprotected resources

**Questions to be considered**

-   Is all your traffic encrypted?
-   Are all endpoints protected by access control?
-   Are your backups encrypted (and signed)?
-   Have you achieved the Cloud Ready belt?
-   Would you know if your AWS costs are unusually high?

**Motivating Values**

-   Coverage of non-functional aspects
-   Tech follows Business
-   Full Ownership and Clear Responsibilities

Inspired by:
------------

-   John Lewis: <http://engineering-principles.onejl.uk/>

-   AutoScout24: <https://github.com/Scout24/scout24-engineering-values-and-principles>

-   Zalando: <https://github.com/zalando/engineering-principles>