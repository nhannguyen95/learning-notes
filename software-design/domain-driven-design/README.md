## Business domain and Subdomains

Business domain is a company's main area of activity, or the service the company provides to its clients.

Subdomains are fine-grained area of business activity and help form a company's business domain.

3 types of subdomains:
- Core subdomains: what gives a company's competitive advantage(s), has high (implementation) complexity, should be done in-house.
- Generic subdomains: what all companies are doing in the same way, has high (implementation) complexity. They are widely known problems that are already solved (e.g. encryption, authn & authz, etc). They should be solved using open source or existing solutions.
- Supporting subdomains: what supports company's business, do not provide any competitive advantage(s), has low (implementation) complexity, can be out-sourced.

As a general guidance on identifying subdomains: subdomains are a set of coherent use cases (involve same actor, business entities, manipulate a closely related set of data, etc.).

## Domain experts

Domain experts are neither the analysts gathering the requirments nor the engineers designing the system, they are the people who identified the business problem in the first place and from whom all business knowledge originates. As a rule of thumb, they are either the people coming up with the requirements or the software's end users.