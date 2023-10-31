---
layout: single
title:  "My Mental Model of Domain-Driven Design"
---
I would like to share my perspective on Domain-Driven Design and what it means to me.

**Why?** Over the past few years, I have been involved in multiple projects focused on rewriting the [Big Ball of Mud](http://www.laputan.org/mud/). During this time, I often found myself questioning the reasons behind these rewrites. What were the initial motivations? Were they driven by architecture, (re)organization, business, or other factors? Nearly 90% of the projects were initially enthusiastic about [Microservices Architecture](https://microservices.io/) (and I must admit, I was caught up in the hype too), but without considering the business context. This made me reflect on why this was happening and I realized that many IT professionals who opted for this architectural approach for their company had misconceptions about what [Microservices Architecture](https://microservices.io/) truly entails.

During my *Software Architect Journey*, I began to approach Software Architecture from a different perspective, considering it not only as code but also as business use cases. This shift in mindset led me to explore [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html) and how to efficiently construct *Distributed Systems*.

First of all. You cannot start implementing [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html) in an organization at the engineering level. As the name of this software design approach implies, it is **Domain-Driven**, meaning that the individuals who are most familiar with the domain are the **business and domain experts, not the software engineers**.

Before implementing anything, it is important to have a **clear understanding of the organization's business** perspective and priorities. This understanding should be shared across all levels of the organization, including business stakeholders, domain experts, software engineers, testers, and others. The **business context** is divided into **multiple bounded contexts**, each representing a specific part of the business process - models, business rules, processes. Each [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html) should have its own unique [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html). There are various techniques, such as [Event Storming](https://www.eventstorming.com/) or [Domain Storytelling](https://domainstorytelling.org/), that can help identify the appropriate boundaries and define the ubiquitous language. Once the Ubiquitous Language is established, it should be used consistently across all communication, documentation, source code, and tests. **It breaks barriers between businesses and engineers and allowing them to work together directly.**

When designing a single [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html), it is important to always focus on the business:

- How can I determine if my context is working effectively?
    - What are the key business metrics?
- What are the domain models, business validation, rules, and processes?
- Which results of actions (domain events) have value?

Other important questions should be guided by domain experts and business stakeholders, as they possess knowledge about the domain.

Only then, and not before, should you consider implementation. It's really *easier, faster and cheaper* to do [Strategy Design of Domain-Driven Design](https://thedomaindrivendesign.io/what-is-strategic-design/) by domain, not by engineers.

*If you want to go far, do things right.*