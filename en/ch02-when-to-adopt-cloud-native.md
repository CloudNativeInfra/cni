# Chapter 2 When to Adopt Cloud Native

Cloud native infrastructure is not for everybody. Any architecture design is a series of trade-offs. Only people familiar with their own requirements can decide what trade-offs are beneficial or detrimental to their needs.

Do not adopt a tool or design without understanding its impact and limitations. We believe cloud native infrastructure has many benefits, but realize it should not be blindly adopted. We would hate to lead someone down the wrong path for their needs.

How can you know if you should pursue architecting with cloud native infrastructure? Here are some questions you can ask to figure out if cloud native infrastructure is best for you:

- Do you have cloud native applications? (See Chapter 1 for application features that can benefit from cloud native infrastructure.)
- Is your engineering team willing and able to write production-quality code that embodies their job functions as software?
- Do you have API-driven infrastructure (IaaS) on-premises or in a public cloud?
- Does your business need faster development cycles or nonlinear people/systems scaling ratios?

If you answered yes to all of these questions, then you will likely benefit from the infrastructure described in the rest of this book. If you answered no to any of these questions, it doesn’t mean you cannot benefit from some cloud native practices, but you may need to do more work before you are ready to fully benefit from this type of infrastructure.

It is just as bad to try to adopt cloud native infrastructure before your business is ready as it is to force a solution that is not right. By trying to gain the benefits without doing due diligence, you will likely fail and write off the architecture as flawed or of no benefit. A tried and failed solution will likely be harder to adopt later, no matter if it is the right solution or not, due to past prejudices.

We will discuss some of the areas to focus on when preparing your organization and technology to be cloud native. There are many things to consider but some of the key areas are your applications, the people at your organization, infrastructure systems, and your business.

## Applications

Applications are the easiest part to get ready. The design patterns are well established, and tooling has improved dramatically since the advent of the public cloud. If you are not able to build cloud native applications and automatically deploy them through a verified and tested pipeline, you should not move forward with adopting the infrastructure to support them.

Building cloud native applications does not mean you must first have microservices. It does not mean you have to be developing all your software in the newest trending languages. It means you have to write software that can be managed by software.

The only interaction humans should have with cloud native applications is during their development.1 Everything else should be managed by the infrastructure or other applications.

Another way to know applications are ready is when they need to dynamically scale with multiple instances. Scaling typically implies multiple copies of the same application behind a load balancer. It assumes that applications store state in a storage service (i.e., database) and do not require complex coordination between running instances.

Dynamic application management implies that a human is not doing the work.Application metrics trigger the scaling, and the infrastructure does the right thing to scale the application. This is a basic feature of most cloud environments. Running autoscaling groups don’t mean you have cloud native infrastructure; but if auto-scaling is a requirement, it may indicate that your applications are ready.

In order for applications to benefit, the people who write the applications and configure the infrastructure need to support this method of working. Without people ready to give up control to software, you’ll never realize the benefits.

## People

People are the hardest part of cloud native infrastructure.

If you want to build an architecture that replaces people’s functions and decisions with software, you need to make sure they know you have their best interests in mind. They need to not only accept the change but also be asking for it and building it themselves.

Developing applications is hard; operating infrastructure is hard. Application developers often believe they can replace infrastructure operators with tooling and automation. Infrastructure operators wish application developers would write more reliable code with automatable debugging and recovery. These tensions are the basis for DevOps, which has many other books written about it, including EffectiveDevOps, by Jennifer Davis and Katherine Daniels (O’Reilly, 2016).

People don’t scale, nor are they good at repetitive, mundane tasks.

The goal of application and systems engineers should be to take away the mundane and repetitive tasks so they can focus on more interesting problems. They need to have the skills to develop software that can contain their business logic and decisions.There need to be enough engineers to write the needed software, and more importantly, maintain it.

The most critical aspect is that they need to work together. One side of engineering can not migrate to a new way of running and managing applications without the support of the other. Team organization and communication structure is important.

We will address team readiness soon, but first, we must decide when infrastructure systems are ready for cloud native infrastructure.

## Systems

Cloud native applications need system abstractions. The application should not be concerned with an individual, hardcoded hostname. If your applications cannot run without careful placement on individual hosts, then your systems are not ready for cloud native infrastructure.

Taking a single server (virtual or physical) running an operating system and turning it into a method by which to access resources is what we mean when we say “abstractions.” Individual systems should not be the target of deployment for an application.Resources (CPU, RAM, and disk) should be pooled across all available machines and then divvied up by the platform from applications’ requests.

In cloud native infrastructure, you must hide underlying systems to improve reliability. Cloud infrastructure, like applications, expects failures of underlying components to occur and is designed to handle such failures gracefully. This is needed because the infrastructure engineers no longer have control of everything in the stack.

> **Is Kubernetes Cloud Native Infrastructure?**
>
> Kubernetes is a framework that makes managing applications easier and promotes doing so in a cloud native way. However, you can also use Kubernetes in a very un-cloud native way.
>
> Kubernetes exposes extensions to build on top of its core functionality, but it is not the end goal for your infrastructure. Other projects (e.g., OpenShift) build on top of it to abstract away Kubernetes from the developer and applications.
>
> Platforms are where your applications should run. Cloud native infrastructure supports them but also encourages ways to run infrastructure.
>
> If your applications are dynamic but your infrastructure is static, you will soon reach an impasse that cannot be addressed with Kubernetes alone.

Infrastructure is ready to become cloud native when it is no longer a challenge. Once infrastructure becomes easy, automated, self-serviceable, and dynamic, it has the potential to be ignored. When systems can be ignored and the technology becomes mundane, it’s time to move up the stack.

If your systems management relies on ordering hardware or running in a “hybrid cloud,” your systems may not be ready. It is possible to manage a data center and becloud native. You will need to be vigilant in separating the responsibilities of building data centers from those of managing infrastructure.

Google, Facebook, Amazon, and Microsoft have all found benefits in creating hardware from scratch through their Open Compute Project. Needing to create their own hardware was a limitation of performance and cost. Because there is a clear separation of responsibilities between hardware design and infrastructure builders, these companies are able to run cloud native infrastructure at the same time they’re creating custom hardware. They are not hindered by running “on-prem.” Instead, they can optimize their hardware and software together to gain more efficiency and performance out of their infrastructure.

Managing your own data center is a big investment of time and money. Creating an on-premises cloud is too. Doing both will require teams to build and manage the data center, teams to create and maintain the APIs, and teams to create abstractions on top of the IaaS APIs.

All of this can be done, and it is up to your business to decide if there is value in managing the entire stack.

Now we can look at what other areas of business need to be prepared for a shift to cloud native practices.

## Business

> If the architecture of the system and the architecture of the organization are at odds, the architecture of the organization wins.
>
> —Ruth Malan, “Conway’s Law”

Businesses are very slow to change. They may be ready to adopt cloud native practices when scaling people to manage scaling systems is no longer working, and when product development requires more agility.

People don’t scale infinitely. For each person added to manage more servers or develop more code, there is a strain on the human infrastructure that supports them(e.g., office space). There is also overhead for other people because there needs to be more communication and more coordination.

As we discussed in Chapter 1, by using a public cloud, you can reduce some of the process and people overhead by renting server time. Even with a public cloud, you will still have people that manage infrastructure details (e.g., servers, services, and user accounts).

The business is ready to adopt cloud native practices when the communication structure reflects the infrastructure and applications the business needs to create. This includes communications structures that mirror architectures like microservices. They could be small, independent teams that do not have to go through layers of management to talk to or work with other teams.

> **DevOps and Cloud Native**
>
> DevOps can complement the way teams work together and influence the type of tool used. It has many benefits for companies that adopt it, including rapid prototyping and increased deployment velocity. It also has a strong focus on the culture of the organization.
>
> Cloud native needs high-performing organizations but focuses on design, architecture, and hygiene more than team workflows and culture. However, if you think you can implement successful cloud native patterns without addressing how your application developers, infrastructure operators, and anyone in the technology department interact, you may be in for a surprise.

Another limiting factor that forces business change is needing more application agility. Businesses need to not only deploy fast but also drastically change what they deploy.

The raw number of deploys does not matter. What matters is providing customer value as quickly as possible. Believing the software deployed will meet all of the customer's needs the first time, or even the 100th time, is a fallacy.

When the business realizes it needs to iterate and change frequently, it may be ready to adopt cloud native applications. As soon as it finds limitations in people efficiency and old process limitations, and it’s open to change, it is ready for cloud native infrastructure.

All the factors that indicate when to adopt cloud native don’t tell the full story. Any design is about trade-offs. So here are some situations when cloud native infrastructure is not the right choice.

## When You Don’t Need Cloud Native Infrastructure

Understanding the benefits of a system is important only when you know the limitations. That is, knowing which limitations are acceptable for your needs can often be more of a deciding factor than the benefits.

It is also important to keep in mind that needs change over time. The functionality that is critical to have now may not be in the future. Likewise, if the following situations don’t make this architecture ideal now, you have control over many of these situations and can change to adopt them.

### Technical Limitations

Just like applications, with infrastructure the easiest items to change are technical. If you know when you should adopt cloud native infrastructure based on technical merit, you can reverse those traits to also find out when you should not adopt cloud native infrastructure.

The first of these limitations is not having cloud native applications. As discussed in

Chapter 1, if your applications need human interaction, whether it be scheduling, restarting, or searching logs, cloud native infrastructure will be of little benefit.

Even having an application that can be dynamically scheduled does not make it cloud native. If your application runs on Kubernetes but still requires humans to set up monitoring, log collection, or a load balancer, it’s not cloud native. And just because you have Kubernetes does not mean you’re done.

If you have an orchestrator, it’s important to look at what it took to get it running. Did you have to place a purchase order, create a ticket, or send an email to get servers?

Those are indicators you don’t have the self-service infrastructure, which is a requirement for a cloud.

In the cloud, you provide nothing more than billing information and call an API.Even if you run servers on-prem, you should have a team that builds IaaS, and then you layer the cloud native infrastructure on top.

If you’re building a cloud environment in your own data center, Figure 2-1 shows an example of where your underlying infrastructure components fit. All raw components (e.g., computer, storage, network) should be available from a self-service IaaS API.

![f-2-1](../images/f-2-1.jpg)

Figure 2-1. Example layers for cloud native infrastructure

In a public cloud, you have IaaS and hosted services, but that doesn’t mean your business is ready for what the public cloud can enable.

When you’re building a platform to run applications, it’s important to know what you are getting into. Initial development is only a small fraction of what it takes to build and maintain a platform, especially one that is business critical.

Maintenance typically consumes about 40 to 80 percent (60 percent on average) of software costs. Therefore, it is probably the most important life cycle phase.2

Discovering the business requirements and building the skills needed to do the development might be too much for a small team. Once you have the skills to develop the needed platform, you still need to invest time to improve and maintain the system.This takes considerably longer than initial development.

Providing the absolute best environment for businesses to run is the product of public cloud providers. If you’re not able or willing to also make your platform a differentiator for your business, then you should not build one yourself.

Keep in mind that you don’t have to build everything yourself. You can use services or products that can be assembled into the platform you need.

Reliability is still a key feature of your infrastructure. If you’re not ready to give up control to lower levels of the infrastructure stack and still make a reliable product through embracing failures, then cloud native infrastructure is not the right choice.

There are also limitations that may be out of your control. The nontechnical limitations are just as important and harder to fix.

### Business Limitations

If existing processes don’t support changing the infrastructure, then you need to surmount that obstacle first. Luckily you do not have to do it alone.

This book will hopefully help clearly explain the benefits and process to people who need convincing. There are also many case studies and companies sharing their experience of adopting these practices. You will find one case study in Appendix C of this book, but it is up to you to find relevant examples for your business and to share them with your peers and management.

If the business doesn’t already have an avenue for experimentation and a culture that supports trying new things (and the consequences that come with failure), then changing processes will likely be impossible. In this case, your options are to reach a critical point where things have to change or to convince management that change is necessary.

It is impossible to tell from the outside if a business is ready to adopt cloud native architecture. Some processes that are clear indicators that the company is not ready to include:

- Resource requests that require human intervention
- Regularly scheduled maintenance windows that require manual labor
- Manual inventory tracking and resource assignments
- Spreadsheet inventory

If people besides the team responsible for a service are involved in the process to schedule, deploy, upgrade, or monitor the service, you may need to address those processes before or during a migration to cloud native infrastructure.

There are also sometimes processes outside of the business’s control, such as industry regulations. Sadly, these are even harder and slower to change than internal processes.

If industry regulations limit the velocity or agility of development, we have no advice, except do what you can. If regulations don’t allow running in a public cloud, try your best to use technologies to run one on-premises. Management will need to make a case for changing regulations to whatever governing body sets them.

There is one other nontechnical hindrance to cloud native infrastructure. In some companies, there is a culture of not using third-party services.3

If your company is not willing, or via a process not able, to use third-party, hosted services, it may not be the right time to adopt cloud native infrastructure. We will discuss when to consume hosted services in more detail in Appendix B.

## Conclusion

> To succeed, planning alone is insufficient. One must improvise as well.—Isaac Asimov

Throughout this chapter, we have discussed considerations of when to adopt cloud native infrastructure. There are many areas to keep in mind and each situation is unique. Hopefully, some of these guidelines can help you discover the opportune time to make the change.

If your company has already been adopting some cloud native practices, these questions can help identify other areas that can also adopt this architecture. It is important to know if the trade-offs and benefits are the right solutions when you should adopt them, and how you can get started.

If you are not already applying cloud native practices to your work, there are no shortcuts. The business and employees will need to collectively decide it is the right solution and make progress together. No one succeeds alone.
