# Appendix B Lock-in

There is much debate about using a cloud provider and avoiding vendor lock-in. The debate is often more ideological than practical.

Lock-in is often a concern for engineers and management. It should be weighed as a risk to an application in the same way as choosing a programming language or framework would be. The selection of a programming language and cloud provider are forms of lock-in, and it is the engineer’s responsibility to understand the risks and know when the risk is acceptable.

There are a few things to keep in mind when you are choosing a vendor or technology:

- Lock-in is unavoidable.
- Lock-in is a risk, but not always high.
- Don’t outsource thinking.

## Lock-in Is Unavoidable

In technology there are two types of lock-in:

**Technology lock-in**

Deals with decisions lower in the stack of development

**Vendor lock-in**

Most often, deals with services and software that are consumed as part of the project (vendor lock-in can also include hardware and operating systems, but we will focus only on services)

### Technology Lock-in

Developers will choose technologies they are familiar with or ones that provide the greatest benefit for the application being developed. These technologies can range from vendor-provided technologies (e.g., .NET and Oracle Database) to open source software (e.g., Python and PostgreSQL).

The lock-in provided at this level often requires conforming to an API or specification that will influence how the application is developed. There are sometimes alternatives to chosen technologies, but they often come with a high switching cost, because the technology influences much about how the application was designed.

### Vendor Lock-in

Vendors, such as cloud providers, are a different form of lock-in. In this case, you are consuming resources from the vendor. This can be infrastructure resources (e.g., compute and storage), or it can be hosted software (e.g., Gmail).

The higher up the stack you consume resources, the more value you should get from the resource being consumed (e.g., Heroku). High-level resources are the most abstracted away from lower-level resources and allow products to be productive faster.

## Lock-in Is a Risk

Technology lock-in is often a one-time decision or agreement with a vendor to use the technology. If you no longer have a support agreement with the vendor, your software does not immediately break; it just becomes self-supported.

Open source software can reduce the amount of lock-in from technology, but it does not eliminate it. Using open standards reduces the lock-in even more, but it’s important to know the difference between open standards and open source.

Just because someone else wrote the code doesn’t make it a standard. Likewise, proprietary systems can form unofficial standards that allow a choice to migrate away from them (e.g., AWS S3).

The reason vendor lock-in is often spoken about more than technology lock-in is because vendor lock-in has a higher risk than technology. If you don’t pay the vendor, your application will cease to run; you no longer have access to the resources you were paying for.

As alluded to earlier, vendor services provide more value because they allow product development to not require all of the lower-level implementations. Don’t avoid hosted services to eliminate risk; you should weigh the risk and reward of the service just as you would anything else.

If the service provides a standard interface, it is a very low risk. The more custom the interface is, or the more unique the product is, the higher the risk it will be to switch.

## Don’t Outsource Thinking

One of the goals of this book is to help you make decisions yourself. Don’t blindly take advice from other people or reports without knowing the context of the advice and how it applies to your situation.

If you can deliver products faster by consuming hosted cloud services, you should pick a vendor and start consuming them. While measuring risk is good, having endless debates over multiple vendors with similar solutions and building the service yourself is not a good use of time.

If multiple vendors provide similar services, pick the one that is easiest to adopt. Limitations will quickly surface after you begin using the service. The biggest factor when picking a vendor is to choose one with the same pace of innovation as you.

If the vendor innovates faster than you, then you will not be able to take advantage of its newest technologies and may have to spend much of your time migrating from older technologies. If the vendor’s innovation is too slow, then you will have to build your own abstractions on top of what the vendor provides, and you will not be focusing on your business objectives.

To stay competitive, you may need to consume resources that do not yet have standards or alternatives (e.g., new and experimental services). Don’t be scared of them just because they may leave you locked into that service. Weigh the risk of staying competitive or losing market share to your competition, who may innovate faster.

Understand what risks you are not able to avoid and what risks are too much for your business. Make decisions to maximize reward and minimize risk.