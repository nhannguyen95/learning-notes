## Business domain and Subdomains

Business domain is a company's main area of activity, or the service the company provides to its clients.

Subdomains are fine-grained area of business activity and help form a company's business domain. Their goal is to *provide solutions* for specific business capabilities.

3 types of subdomains:
- Core subdomains: what gives a company's competitive advantage(s), has high (implementation) complexity, should be done in-house.
- Generic subdomains: what all companies are doing in the same way, has high (implementation) complexity. They are widely known problems that are already solved (e.g. encryption, authn & authz, etc). They should be solved using open source or existing solutions.
- Supporting subdomains: what supports company's business, do not provide any competitive advantage(s), has low (implementation) complexity, can be out-sourced.

As a general guidance on identifying subdomains: subdomains are a set of coherent use cases (involve same actor, business entities, manipulate a closely related set of data, etc.).

## Domain experts

Domain experts are neither the analysts gathering the requirments nor the engineers designing the system, they are the people who identified the business problem in the first place and from whom all business knowledge originates. As a rule of thumb, they are either the people coming up with the requirements or the software's end users.

## Ubiquitous language

Software development is a learning process; working code is a side effect. A software project's success depends on the effectiveness of knowledge sharing between domain experts and software engineers. And to do so effectively, there's need to be an ubiquitous language.

Uniquitous language should be divided into multiple languages and each should be assigned to bounded context to avoid ambiguities in communication. E.g., the term *lead* can have different meanings in marketing and sales department, we can create *marketing context* and *sales context*, in which the term *lead* exists with its own meaning.

The difference between subdomains and bounded contexts is that the former are discovered (by the business strategy), while the later are designed (to address specific project's context and constraints). Bounded context can have a 1-1 or 1-many mapping to subdomains.

**Model**: a model is a *simplified* representation of a thing/phenomenon that intentinally emphasizes certain aspects while ignoring others. A model is *NOT* (or does *NOT* try to be) a copy of real world, instead it contains only the details needed to fulfill its purpose. For example, a subway map model does not represents all the details of our planet, instead it only shows subway data. The purpose of such abstraction is *NOT* to be vague (by removing redundant details), but to create a new sementic level in which one can be absolutely precise.