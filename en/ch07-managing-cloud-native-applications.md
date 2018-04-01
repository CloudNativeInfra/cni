# Chapter 7 Managing Cloud Native Applications

Cloud native applications are designed to be maintained by infrastructure. As we've shown in the previous chapters, cloud native infrastructure is designed to be maintained by applications.

With traditional infrastructure, the majority of the work schedule, maintain and upgrade applications are done by humans. This can include manually running services on individual hosts or defining a snapshot of infrastructure and applications in automation tools.

But if infrastructure can be managed by applications and at the same time manage the applications, then infrastructure tooling becomes just another application. The responsibilities engineers have with infrastructure can be expressed in the reconciler pattern and built into applications that run on that infrastructure.

We just spent the past three chapters explaining how we build applications that can manage infrastructure. This chapter will address how to run those applications, or any application, on the infrastructure.

As discussed earlier, it’s important to keep your infrastructure and applications simple. A common way to manage complexity in applications is to break them apart into small, understandable components. We usually accomplish this by creating single-purpose services,1 or breaking out code into a series of event-triggered functions.2

The proliferation of smaller, deployable units can be overwhelming for even the most automated infrastructure. The only way to manage a large number of applications is to have them take on the operational functionally described in Chapter 1. The applications need to become cloud native before they can be managed at scale.

This chapter will not help you build the next great app, but it should give you some starting points in making your applications work well when running on cloud native infrastructure.

## Application Design

There are many books that discuss how applications should be architected. This book is not intended to be one of them. However, it is still important to understand how application architecture influences effective infrastructure designs.

As we discussed in Chapter 1, we are going to assume the applications are designed to be cloud native because they gain the most benefits from cloud native infrastructure.Fundamentally, cloud native means the applications are designed to be managed by software, not humans.

The design of an application is a separate consideration from how it is packaged.Applications can be cloud native and packaged as an RPM or DEB files and deployed to VMs instead of containers. They can be monolithic or microservices, written in Java or Go.

These implementation details do not make an application designed to run in the cloud.

As an example, let’s pretend we have an application written in Go and packaged in a container. We can even say the container runs on Kubernetes and would be considered a microservice by whatever definition you choose.

Is this pretend application “cloud native”?

What if the application logs all activity to a file and hardcodes the database IP address? Maybe it doesn’t accept runtime configuration and stores state on the local disk. What if it doesn’t exist in a predictable manner, or hangs and waits for a human to debug it?

This pretends application may appear cloud native from the chosen language and packaging, but it is most definitely not. A framework such as Kubernetes can help manage this application through various features, but even if you’re able to make it run, the application is clearly designed to be maintained and run by humans.

Some of the features that make an application run better on cloud native infrastructure are explained in more detail in Chapter 1. If we have the features prescribed in chapter 1, there is still another consideration for applications: how do we effectively manage them?

## Implementing Cloud Native Patterns

Patterns such as resiliency, service discovery, configuration, logging, health checks, and metrics can all be implemented in applications in different ways. A common practice to implement these patterns is through standard language libraries imported into the applications. Netflix OSS and Twitter’s Finagle are very good examples of implementing these features in Java language libraries.

When you use a library, an application can import the library, and it will automatically get many of these features without extra code. This model makes a lot of sense when there are few supported languages within an organization. It allows best practice to be the easy thing to do.

When organizations start implementing microservices, they tend to drift toward polyglot services.3 This allows for freedom to choose the right language for the service but makes it very difficult to maintain libraries for all the languages.

Another way to get some of these features is through what is known as the “sidecar” pattern. This pattern bundles processes with applications that implement the various management features. It is often implemented as a separate container, but you can also implement it by just running another daemon on a VM.

Examples of sidecars include the following:

**Envoy proxy**

Adds resiliency and metrics to services

**Registrator**

Registers services with an external service discovery

**Configuration**

Watches for configuration changes and notifies the service process to reload

**Health endpoint**

Provide HTTP endpoints for checking the health of the application

Sidecar containers can even be used to adapt polyglot containers to expose language-specific endpoints to interact with applications that use libraries. Prana from Netflix does just that for applications that don’t use their standard Java library.

Sidecar patterns make sense when centralized teams manage specific sidecar pro‐cesses. If an engineer wants to expose metrics in their service, they can build it into the application—or a separate team can also provide a sidecar that processes logging output and exposes the calculated metrics for them.

In both cases, the service can have functionality added with less effort than rewriting the application. Once the ability to manage the application with software is available, let’s look at how the application’s lifecycle should be managed.

## Application Life Cycle

Life cycles for cloud native applications are no different than traditional applications, except their stages should be managed by software.

This chapter is not intended to explain all the patterns and options involved in managing applications. We will briefly discuss a few stages that particularly benefit from running cloud native applications on top of cloud native infrastructure: deploy, run, and retire.

These topics are not all inclusive of every option, but many other books and articles exist to explore the options, depending on the application’s architecture, language, and chosen libraries.

### Deploy

Deployments are one area where applications rely on infrastructure the most. While there is nothing stopping an application from deploying itself, there are still many other aspects that the infrastructure manages.

How you do integration and delivery are topics we will not address here, but a few practices in this space are clear. Application deployment is more than just taking the code and running it.

Cloud native applications are designed to be managed by software in all stages. This includes ongoing health checks as well as initial deployments. Human bottlenecks should be eliminated as much as possible in the technology, processes, and policies.

Deployments for applications should be automated, self-service, and, if under active development, frequent. They should also be tested, verified, and uneventful.

Replacing every instance of an application at once is rarely the solution for new versions and features. New features are “gated” behind configuration flags, which can be selectively and dynamically enabled without an application restart. Version upgrades are partially rolled out, verified with tests, and, when all tests pass, rolled out in a controlled manner.

When new features are enabled or new versions deployed, there should exist mechanisms to control traffic toward or away from the application (see Appendix A). This can limit outage impact and allows slow rollouts and faster feedback loops for application performance and feature usage.

The infrastructure should take care of all details of deploying software. An engineer can define the application version, infrastructure requirements, and dependencies, and the infrastructure will drive toward that state until it has satisfied all requirements or the requirements change.

### Run

Running the application should be the most uneventful and stable stage of an application's lifecycle. The two most important aspects of running software are discussed in chapter 1: observability to understand what the application is doing, and operability to be able to change the application as needed.

We already went into detail in Chapter 1 about observability for applications by reporting health and telemetry data, but what do you do when things don’t work as intended? If an application’s telemetry data says it’s not meeting the SLO, how can you troubleshoot and debug the application?

With cloud native applications, you should not SSH into a server and dig through logs. It may even be worth considering if you need SSH, log files, or servers at all.

You still need application access (API), log data (cloud logging), and servers somewhere in the stack, but it’s worth going through the exercise to see if you need the traditional tools at all. When things break, you need a way to debug the application and infrastructure components.

When debugging a broken system, you should first look at your infrastructure tests, as explained in Chapter 5. Testing should expose any infrastructure components that are not configured properly or are not providing the expected performance.

Just because you don’t manage the underlying infrastructure doesn’t mean the infrastructure cannot be the cause of your problems. Having tests to validate expectations will ensure your infrastructure is performing how you expect.

After infrastructure has been ruled out, you should look to the application for more information. The best places to turn for application debugging are application performance management (APM) and possibly distributed application tracing via standards such as OpenTracing.

OpenTracing examples, implementation, and APM are out of scope for this book. Asa very brief overview, OpenTracing allows you to trace calls throughout the application to more easily identify network and application communication problems. An example visualization of OpenTracing can be seen in Figure 7-1. APM adds tools to your applications for reporting metrics and faults to a collection service.

![f-7-1](../images/f-7-1.jpg)

Figure 7-1. OpenTracing visualization

When tests and tracing still do not expose the problem, sometimes you just need to enable more verbose logging on the application. But how do you enable debugging without destroying the problem?

Configuration at runtime is important for applications, but in a cloud native environment, the configuration should be dynamic without application restarts. Configuration options are still implemented via a library in the application, but flag values should have the ability to dynamically change through a centralized coordinator, application API calls, HTTP headers, or a myriad of ways.

Two examples of dynamic configuration are Netflix’s Archaius and Facebook’s Gate‐keeper. Justin Mitchell, a former Facebook engineering manager, shared in a Quora post that:

> [Gatekeeper] decoupled feature releasing from code deployment. Features might be released over the course of days or weeks as we watched user metrics, performance, and made sure services were in place for it to scale.

Allowing dynamic control over application configuration enables more control overexposed features and better test coverage of deployed code. Just because pushing new code is easy doesn’t mean it is the right solution for every situation.

Infrastructure can help solve this problem and enable more flexible applications by coordinating when features are enabled and routing traffic based on advanced network policies. This pattern also allows finer-grained controls and better coordination of roll-out or roll-back scenarios.

In a dynamic, self-service environment the number of applications that will get deployed will grow rapidly. You need to make sure you have an easy way to dynamically debug applications in a similar self-service model to deploy the applications.

As much as engineers love pushing new applications, it is conversely difficult to get them to retire old applications. Even still, it is a crucial stage in an application’s lifecycle.

### Retire

Deploying new applications and services is common in a fast-moving environment.Retiring applications should be self-service in the same way as creating them.

If new services and resources are deployed and monitored automatically, they should also be retired under the same criteria. Deploying new services as quickly as possible with no removal of unused services is the easiest way to accrue technical debt.

Identifying services and resources that should be retired is business specific. You can use empirical data from your application’s telemetry measurements to know if an application is being used, but the decision to retire applications should be made by the business.

Infrastructure components (e.g., VM instances and load balancer endpoints) should be automatically cleaned up when not needed. One example of automatic component cleanup is Netflix’s Janitor Monkey. The company explains in a blog post:

> Janitor Monkey determines whether a resource should be a cleanup candidate by applying a set of rules on it. If any of the rules determines that the resource is a cleanup candidate, Janitor Monkey marks the resource and schedules a time to clean it up.

The goal in all of these application stages is to have infrastructure and software manage the aspects that would traditionally be managed by a human. Instead of writing automation scripts that run once by a human, we employ the reconciler pattern combined with component metadata to constantly run and make decisions about actions that need to be taken on a high level based on current context.

Application lifecycle stages are not the only aspects where applications depend on infrastructure. There are also some fundamental services for which applications in every stage will depend on infrastructure. We will look at some of the supporting services and APIs infrastructure provides to applications in the next section.

## Application Requirements on Infrastructure

Cloud native applications expect more from infrastructure than just executing binary. They need abstractions, isolations, and guarantees about how they’ll run and be managed. In return, they are required to provide hooks and APIs to allow the infrastructure to manage them. To be successful, there needs to be a symbiotic relationship.

We defined cloud native applications in Chapter 1, and just discussed some life cycle requirements. Now let’s look at more expectations they have from an infrastructure built to run them:

- Runtime and isolation
- Resource allocation and scheduling
- Environment isolation
- Service discovery
- State management
- Monitoring and logging
- Metrics aggregation
- Debugging and tracing

All of these should be default options for services or provided from self-service APIs.We will explain each requirement in more detail to make sure the expectations are clearly defined.

### Application Runtime and Isolation

Traditional applications only needed a kernel and possibly an interpreter to run.Cloud native applications still need that, but they also need to be isolated from the operating system and other applications where they run. Isolation enables multiple applications to run on the same server and control their dependencies and resources.

Application isolation is sometimes called multitenancy. That term can be used for multiple applications running on the same server and for multiple users running applications in a shared cluster. The users can be running verified, trusted code or they may be running code you have no control over and do not trust.

To be cloud native does not require using containers. Netflix pioneered many of the cloud native patterns, and when the company transitioned to running on a public cloud, it used VMs as their deployment artifact, not containers. FaaS services (e.g., AWS Lambda) are another popular cloud native technology for packaging and deploying code. In most cases, they use containers for application isolation but the container packaging is hidden from the user.

> **What Is a Container?**
>
> There are many different implementations of containers. Docker popularized the term container to describe a way to package and run an application in an isolated environment. Fundamentally, containers use kernel primitives or hardware features to isolate processes on a single operating system.
>
> Levels of container isolation can vary, but usually, it means the application runs with an isolated root filesystem, namespaces, and resource allocation (e.g., CPU and RAM)from other processes on the same server. The container format has been adopted by many projects and has created the Open Container Initiative (OCI), which defines standards on how to package and run an application container.

Isolation also puts a burden on the engineers writing the application. They are now responsible for declaring all software dependencies. If they fail to do so, the application will not run because necessary libraries will not be available.

Containers are often chosen for cloud native applications because better tooling, processes, and orchestration tools have emerged for managing them. While containers are currently the easiest way to implement runtime and resource isolation, this has not (and likely will not) always be the case.

### Resource Allocation and Scheduling

Historically, applications would provide rough estimates around minimum system requirements, and it was the responsibility of a human to figure out where the application could run.4 Human scheduling can take a long time to prepare the operating system and dependencies for the application to run.

The deployment can be automated through configuration management and provisioning, but it still requires a human to verify resources and tag a server to run the application. Cloud native infrastructure relies on dependency isolation and allows applications to run wherever resources are available.

With isolation, as long as a system has available processing, storage, and access to dependencies, applications can be scheduled anywhere. Dynamic scheduling removes the human bottleneck from making decisions that are better left to machines. A cluster scheduler gathers resource information from all systems and figures out the best place to run the application.

> Having humans schedule application placement doesn’t scale.Humans get sick, take vacations (or at least they should), and are generally bottlenecks. As scale and complexity increases, it also becomes impossible for a human to remember where applications are running.
>
> Many companies try to scale by hiring more people. This exacerbates the problem because then scheduling needs to be coordinatedbetween, multiple people. Eventually, human scheduling resorts to keeping a spreadsheet (or similar solution) of where each application runs.

Dynamic scheduling doesn’t mean operators have no control. There are still ways an operator can override or force a scheduling decision based on knowledge the scheduler may not have. Overrides and manual resource scheduling should be provided via an API and not a meeting request.

Solving these problems is one of the main reasons Google wrote its internal cluster scheduler named Borg. In the Borg research paper, Google points out that:

> Borg provides three main benefits: it (1) hides the details of resource management and failure handling so its users can focus on application development instead; (2) operates with very high reliability and availability, and supports applications that do the same; and (3) lets us run workloads across tens of thousands of machines effectively.

The role of a scheduler in any cloud native environment is much of the same. Fundamentally, it needs to abstract away the many machines and allow users to request resources, not servers.

### Environment Isolation

When applications are made of many services, infrastructure needs to provide a way to have defined isolation with all dependencies. Separating dependencies is traditionally managed by duplicating servers, networks, or clusters into development or testing environments. Infrastructure should be able to logically separate dependencies through application environments without full cluster duplication.

Logically splitting environments allows for better utilization of hardware, less duplication of automation, and easier testing for the application. On some occasions, a separate testing environment is required (e.g., when low-level changes need to be made). However, application testing is not a situation where a full duplicate infra‐structure should be required.

Environments can be traditional permanent dev, test, stage, and production, or they can be a dynamic branch or commit-based. They can even be segments of the production environment with features enabled via dynamic configuration and selective routing to the instances.

Environments should consist of all the data, services, and network resources needed by the application. This includes things such as databases, file shares, and any external services. Cloud native infrastructure can create environments with very low overhead.

Infrastructure should be able to provision the environment however it’s used. Applications should follow best practices to allow flexible configuration to support environments and discover the endpoints for supporting services through service discovery.

### Service Discovery

Applications almost certainly depend on one or more services to provide business benefit. It is the responsibility of the infrastructure to provide a way for services to find each other on a per-environment basis.

Some service discovery requires applications to make an API call, while others do it transparently with DNS or network proxies. It does not matter what tool is used, but it's important that services use service discovery.

While service discovery is one of the oldest network services (i.e., ARP and DNS), itis often overlooked and not utilized. Statically defining service endpoints in a per-instance text file or in the code are not scalable and not suitable for a cloud native environment. Endpoint registration should happen automatically when services are created and endpoints become available or go away.

Cloud native applications work together with the infrastructure to discover their dependent services. These include, but are not limited to, DNS, cloud metadata services, or standalone service discovery tools (i.e., etcd and consul).

### State Management

State management is how infrastructure can know what needs to be done, if anything, to an application instance. This is distinctly different from application lifecycle because the life cycle applies to applications throughout their development process.States apply to instances as they are started and stopped.

It is the application’s responsibility to provide an API or hook so it can check for its current state. The infrastructure’s responsibility is to monitor the instance’s current state and act accordingly.

The following are some application states:

- Submitted
- Scheduled
- Ready
- Healthy
- Unhealthy
- Terminating

A brief overview of the states and corresponding actions would be as follows:

1. An application is submitted to be run.
2. The infrastructure checks the requested resources and schedules the application.
   While the application is starting, it will provide a ready/not ready status.
3. The infrastructure will wait for a ready state and then allow for consumption of the application's resources (e.g., adding the instance to a load balancer).
   If the application is not ready before a specified timeout, the infrastructure will terminate it and schedules a new instance of the application.
4. Once the application is ready, the infrastructure will watch the liveness status and wait for an unhealthy status or until the application is set to no longer run.

There are more states than those listed. States need to be supported by the infrastructure if they are to be correctly checked and acted upon. Kubernetes implements application state management through events, probes, and hooks, but every orchestration platform should have similar application management capabilities.

A Kubernetes event is triggered when an application is submitted, scheduled, or scaled. Probes are used to check when an application is ready to serve traffic (readiness) and to make sure an application is healthy (liveness). Hooks are used for events that need to happen before or after processes start.

The state of an application instance is just as important as application lifecycle management. Infrastructure plays a key role in making sure instances are available and acting on them accordingly.

### Monitoring and Logging

Applications should never have to request to be monitored or logged; they are basic assumptions for running on the infrastructure. More importantly, the configuration for monitoring and logging, if required, should be declarative as code in the same way that application resource requests are made. If you have all the automation to deploy applications but can’t dynamically monitor services, there is still work to be done.

State management (i.e., process health checks) and logging deal with individual instances of an application. The logging system should be able to consolidate logs base on the applications, environments, tags, or any other useful metadata.

Applications should, as much as possible, not have single points of failure and should have multiple instances running. If an application has 100 instances running, the monitoring system should not trigger an alert if a single instance becomes unhealthy.

Monitoring looks holistically at applications and is used for debugging and verifying desired states.5 Monitoring is different than alerting because alerting should be triggered based on the metrics and SLO of the applications.

### Metrics Aggregation

Metrics are required to know how applications behave when they’re in a healthy state.They also can provide insight into what may be broken when they are unhealthy—and just like monitoring, metrics collecting should be requested as code as part of an application definition.

The infrastructure can automatically gather metrics around resource utilization,6 but it is the application’s responsibility to present metrics for service-level indicators.

While monitoring and logging are application health checks, metrics provide the needed telemetry data. Without metrics, there is no way of knowing if the application is meeting service-level objectives to provide business value.

> It may be tempting to pull telemetry and health check data from logs, but be careful, because logging requires post-processing and more overhead than application-specific metric endpoints.
>
> When it comes to gathering metrics, you want as close to real-time data as possible. This requires a simple and low-overhead solution that can scale.
>
> Logging should be used for debugging, and a delay for data processing should be expected.

Similarly to logging, metrics are usually gathered at an instance level and then composed together in aggregate to provide a view of a complete service instead of individual instances.

Once applications present a way to gather metrics, it is the infrastructure’s job to scrape, consolidate, and store the metrics for analysis. Endpoints for gathering metrics should be configurable on a per-application basis, but the data formatting should be standardized so all metrics can be viewed in a single system.

### Debugging and Tracing

Applications are easy to debug during development. Integrated development environments (IDE), code breakpoints, and running in debug mode are all tools the engineer has at his disposal when writing code.

Introspection is much more difficult for deployed applications. This problem is more acute when applications are composed of tens or hundreds of microservices or independently deployed functions. It may also be impossible to have tooling built into applications when services are written in multiple languages and by different teams.

The infrastructure needs to provide a way to debug a whole application and not just the individual services. Debugging can sometimes be done through the logging system, but reproducing bugs requires a shorter feedback loop.

Debugging is a good use of dynamic configuration, discussed earlier. When issues are found, applications can be switched to verbose logging, without restarting, and traffic can be routed to the instances selectively through application proxies.

If the issue cannot be resolved through log output, then distributed tracing provides a different interface to visualize what is happening. Distributed tracing systems such as OpenTracing can complement logs to help humans debug problems.

Tracing provides shorter feedback loops for debugging distributed systems. If it can not be built into applications, it can be done transparently by the infrastructure through proxies or traffic analysis. When you are running any coordinated applications at scale, it is a requirement that the infrastructure provides a way to debug applications.

While there are many benefits and implementation details for setting up tracing in a distributed system, we will not discuss them here. Application tracing has always been important and is increasingly difficult in a distributed system. Cloud native infrastructure needs to provide tracing that can span multiple services in a transparent way.

## Conclusion

The applications requirements have changed: a server with an operating system and package manager is no longer enough. Applications now require coordination of services and higher levels of abstraction. The abstractions allow resources to be separated from servers and consumed programmatically as needed.

The requirements laid out in this chapter are not all the services that infrastructure can provide, but they are the basis for what cloud native applications expect. If the infrastructure does not provide these services, then applications will have to implement them, or they will fail to reach the scale and velocity required by modern business.

Infrastructure won’t evolve on its own; people need to change their behavior and fundamentally think of what it takes to run an application a different way. Luckily there are projects that build on experience from companies that have pioneered these solutions.

Applications depend on the features and services of infrastructure to support agile development. Infrastructure requires applications to expose endpoints and integrations to be managed autonomously. Engineers should use existing tools when possible and build with the goal of designing resilient, simple solutions.