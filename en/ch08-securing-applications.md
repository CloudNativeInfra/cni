# Chapter 8 Securing Applications

We’ve already discussed that infrastructure should only be provisioned from cloudnative applications. We know that the infrastructure is also responsible for running those same applications.

Running infrastructure provisioned and controlled by applications allows us to scale more easily. We scale our infrastructure by learning how to scale applications. We also secure our infrastructure by learning how to secure applications.

In a dynamic environment, we have shown that humans cannot scale to manage the complexity. Why would we think they can scale to handle the policy and security?

This means, just as we had to create the applications that enforce infrastructure state through the reconciler pattern, we need to create applications that enforce security policy. Before we create the applications to enforce policy, we need to write our policy in a machine-parseable format.

## Policy as Code

Policy is hard to put into code because it does not have clearly defined technical implementations. It deals more with how business gets done more than what run a business.

Both the how and the what change frequently, but the how is much more opinionated and not easily abstracted. It also is organization specific and will likely need to know specific details about the communication structure of the people who create the infrastructure.

Policy needs to apply to multiple stages of the application lifecycle. As we discussed in Chapter 7, applications generally have three stages; deploy, run, and retire.

The deploy stage will have a policy applied before the application and infrastructure changes go out. This will include deployment rules and conformity tests. The run stage will include ongoing compliance and enforcement of access controls and segregation. The retire stage is important to make sure no services are left behind in unpatched or unmaintained states.

In each of these stages, you need to break down policies into a definitive, actionable implementation. Vague policy cannot be enforced. You will need to put the implementation in code and then create the application, or use an existing application, to enforce the policy rules.

Once you have policy as code, you should treat it as code. Policy changes should be treated as application changes and tracked in version control.

The same policy that gates your application deployments should also apply to your new policy deployments. The more components of infrastructure you can have tracked and deployed with the same tools as deploying applications, the easier it will be to understand what is running and how changes can impact the system.

A great benefit about policy as the code is that you can easily add or remove policies, and they are tracked, so there’s a record of who did it, when they did it, as well as comments around the commits and pull request. Because the policy exists as code, you can now write tests for your policies! If you want to validate that a policy will do the right thing, you can use the testing practices from Chapter 5.

Let’s look more closely at applying policy to application life cycles.

### Deployment Gating

Deployment gating makes sure how applications get deployed conform to the business rules. This means you will need to build a deployment pipeline and not allow direct-to-production deployments from users’ machines.

You will need to centralize control before enforcing centralized policy, but you should start small and prove that the solution works before implementing it everywhere. The benefits of a deployment pipeline go well beyond policy enforcement and should be standard in any organization with more than a handful of developers.

Here are some examples of policy:

- Deployments can happen only after all tests have passed.
- New applications require a senior developer to review the changes and comment on the pull request.
- Production artifact pushes can happen only from the deployment pipeline.

Gating is not supposed to enforce the running state or API requests to applications. The application should know how infrastructure components are provisioned and policy is applied to those components through compliance and auditing and not during application deployment.

An example of a deployment gating policy is if your organization doesn’t allow code deploys past 3 p.m. on a Friday without manager approval.

This one is pretty easy to put into code. Figure 8-1 is a very simplified diagram to represent the policy.

![f-8-1](../images/f-8-1.jpg)

Figure 8-1. Deployment policy

You can see the policy simply checks the time and day of the week when a deployment is allowed to be deployed. If it is Friday and after 3 p.m., then the policy checks for a manager OK.

The policy can get the OK notification via a verified email sent by the manager, an authenticated API call, or various other means.2 It is up to the policy to decide what the preferred communication method is and how long to wait for approval.

This simple example can be expanded with lots of different options, but it’s important to make sure the policy is not up to humans to parse and enforce. Human interpretation differs, and unclear policies often don’t get enforced at all.

By making sure the policy will block new deployments, we can save ourselves a lot of work in troubleshooting the state of the production environment. Having a chain of events that can be verified in software goes a long way in understanding your systems. Version control and continuous deployment pipelines can verify code; using policy and process as code can verify how and when software is deployed.

Beyond just making sure the right thing gets deployed via company policy, we should also make it easy to deploy the supported thing with templates and enforce them with conformity tests.

### Conformity Testing

In any infrastructure, you need to provide a recommended way to create specific types of applications. These recommendations become building blocks for users to consume and piece together depending on their needs.

It’s important that the recommendations are compostable but not too small. They need to be understandable in their intended functionality and self-service. We've already recommended that cloud native applications be packaged as containers and consumed via an orchestrator; it is up to you to figure out what works best for your users and what components you want to provide.3

> There are multiple ways you can provide templated infrastructure to users.
>
> Some examples are to provide a templated code such as Jsonnet, or fully templated applications such as charts with Helm.
>
> You also could provide templates via your deployment pipeline.These can be Terraform modules or deployment-specific tools such as Spinnaker templates.
>
> Creating deployment templates allows users to become consumers of the template, and as best practices evolve, the users automatically benefit.

The most critical aspect of infrastructure templates is to make it easy to do the right thing and hard to do the wrong thing. If you meet the customer’s need, it will be much easier to get adapters.

However, the whole reason we need templated infrastructure is so we can enforce conformity tests. Conformity tests exist to make sure applications and infrastructure components adhere to the organization’s standards.

Some examples of testing for infrastructure that does not adhere to standards are:

- Services or endpoints not in an automatic scaling group
- Applications not behind a load balancer
- Frontend tiers talking directly to databases

These standards are information about infrastructure that you can find by making a call to the cloud provider’s API. Conformity tests should be run continuously and enforce the architectural standards your company has adopted.

If infrastructure components or application architecture are found to violate the provided standards, they should be terminated as early as possible. The sooner you can codify your templates, the earlier you can check for applications that don’t conform to your standards. It’s important to address unsupported architecture early in the application’s life so reliance on the architectural decisions can be minimized.

Conformity deals with how applications are built and maintaining operability. Compliance testing makes sure the application and infrastructure stay secure.

### Compliance Testing

Compliance testing does not test architectural designs, but rather it focuses on the implementation of components to make sure they adhere to defined policies. The easiest policies to put in the code are those that aid with security. Policies around organizational requirements (e.g., HIPAA) should also be defined as code and tested during compliance testing.

  Some examples of compliance policies are:

- Object storage has limited user access and is not readable or writable by the public internet
- API endpoints all use HTTPS and have valid certificates
- VM instances (if you have any) do not have overly permissive firewall rules

While these policies do not make applications free from all exploits or bugs, compliance policies should minimize the scope of impact if an application does get exploited.

Netflix explains the purpose of its implementation of compliance testing via its Security Monkey in its blog post “The Netflix Simian Army”:

> Security Monkey is an extension of Conformity Monkey. It finds security violations or vulnerabilities, such as improperly configured AWS security groups, and terminates the offending instances. It also ensures that all our SSL and DRM certificates are valid and are not coming up for renewal.

Putting your policy into the code and continue running it by watching the cloud provider API allows you to stay more secure, catch insecure settings immediately, and track the policy over time with version control systems. The model of continually testing your infrastructure against these policies also fits nicely with the reconciler pattern.

If you think of policy as a type of configuration that needs to be applied and enforced, then it can be much simpler to implement it. It’s important to keep in mind that as your infrastructure and business needs change, so should your compliance policies.

Deployment testing watches applications before they are deployed to the infrastructure, and conformity and compliance testing both deal with running applications. The last application lifecycle phase to make sure you have a policy for is knowing when and how to retire applications and lifecycle components.

### Activity Testing

Conformity and compliance testing should remove applications and infrastructure that fail the defined policies. There should also be an application that cleans up old and unused infrastructure components. High-level usage patterns should be based on application telemetry data, but there are still other infrastructure components that are easily forgotten and need to be retired.

In a cloud environment, you can consume resources on demand, but it is all too easy to forget what you demanded. Without automatic cleanup of old or unused resources, you will end up with a big surprise on your usage bill or need to spend costly human hours to do manual auditing and cleanup.

Some examples of resources you should test for and automatically clean up include:

- Old disk snapshots

- Test environments
- Previous application versions

The application responsible for cleanup needs to do the right thing according to the policy by default and provide flexibility for exceptions to be specified by engineers.

As mentioned in Chapter 7, Netflix has implemented what it calls its “Janitor Monkey,” whose implementation perfectly describes this needed pattern:

> Janitor Monkey works in a process of “mark, notify, and delete”. When Janitor Monkey marks a resource as a cleanup candidate, it schedules a time to delete the resource. The delete time is specified in the rule that marks the resource.
>
> Every resource is associated with an owner email, which can be specified as a tag on the resource or you can quickly extend Janitor Monkey to obtain the information from your internal system. The simplest way is using a default email address, e.g. your team’s email list for all the resources. You can configure a number of days for specifying when to let Janitor Monkey send notifications to the resource owner before the scheduled termination. By default, the number is 3, which means that the owner will receive a notification 3 business days ahead of the termination date.
>
> During the 3 day period, the resource owner can decide if the resource is OK to delete.In case a resource needs to be retained longer the owner can use a simple REST interface to flag the resource as not being cleaned by Janitor Monkey. The owner can always use another REST interface to remove the flag and Janitor Monkey will then be able to manage the resource again.
>
> When Janitor Monkey sees a resource marked as a cleanup candidate and the scheduled termination time is already passed, it will delete the resource. The resource owner can also delete the resource manually if he/she wants to release the resource earlier to save cost. When the status of the resource changes which makes the resource not a cleanup candidate, e.g. a detached EBS volume is attached to an instance, Janitor Monkey will unmark the resource and no termination will happen.

Having an application that automatically cleans up your infrastructure will keep your complexity and costs lower. This testing implements the reconciler pattern to the last life cycle stage of our applications.

There are still other practices with infrastructure that are important to consider.Some of them apply to traditional infrastructure, but in a cloud native environment need to be handled differently.

Continually testing aspects of the infrastructure helps you to know you’re adhering to your policies. When infrastructure comes and goes frequently, it is difficult to audit what changes may be responsible for an outage or to use historical data to predict future trends.

If you expect to get that information from billing statements or by extrapolating current snapshot of the infrastructure, you will discover quickly that the information provided there is the wrong tool for the job. For tracking changes and predicting the future, we need to have auditing tools that can quickly provide us the information we need.

## Auditing Infrastructure

Auditing cloud native infrastructure in this sense is not the same as auditing components in the reconciler pattern, nor is it the same as the testing frameworks discussed in Chapter 6. Instead, when we talk about auditing, we mean a high-level overview of change and component relations within the infrastructure.

Keeping track of what existed in the infrastructure and how it relates to other components gives us important context when trying to understand the current state. When something breaks, the first question is almost always, “What changed?” Auditing answers that question for us and can be leveraged to tell us what will be affected if we apply a change.

In a traditional infrastructure, the configuration management database (CMDB) was often the source of truth for the current state of the infrastructure. However, CMDBsdo not track historical versions of the infrastructure or asset relationships.

Cloud providers can give you CMDB replacements via their inventory APIs, but they may not be incentivized to show you historical trends or let you query on specific details you need for troubleshooting, such as hostnames.

A good cloud auditing tool will allow you to show the difference (“diff ”) of your current infrastructure compared to yesterday’s or last week’s. It should be able to combine cloud provider data with other sources (e.g., container orchestrator) to allow you to query the infrastructure for data the cloud provider may not have, and ideally, it can automatically build topographical representations of component relationships.

If your applications run entirely on a single platform (e.g., Kubernetes), it is much easier to gather topology information for resource dependencies. Another way to automatically visualize relationships is by unifying the layer at which relationships happen.

For services running in the cloud, relationships can be identified in the network communication between the services. There are many ways to identify the network traffic between services, but the important consideration with auditing is keeping historical track of the information. You need to be able to easily identify when relationships change just as easily as you identify when components come and go.

> **Methods for Automatically Identifying Service Relationships**
>
> Tracking network traffic between services means you need a tool that is aware of the network protocols used for communication. You cannot simply depend on raw packet traffic flowing in or out of a service endpoint. You’ll need a way to tap into the flow of information to build the relationship model.
>
> A popular way to inspect the network traffic and build a dependency graph is through network proxies.
>
> Some implementation examples for network proxies are linkerd and envoy. Services are tooled to flow all traffic through these proxies, which are aware of protocols being used and other dependent services. Proxies also allow for other network resiliency patterns, as discussed in Appendix B.

Tracking topology in relation to time is a powerful auditing tool. Combined with infrastructure history it will make sure the “what changed?” question is easier to answer.

There is also an aspect of auditing that deals with building trust in the current state of the infrastructure. Applications gain trust through the testing tools described, but some aspects of infrastructure cannot have the same tests applied.

Building infrastructure with verifiable, reproducible components can provide a great deal of trust. This practice is known as immutable infrastructure.

## Immutable Infrastructure

Immutable infrastructure is the practice of creating change through replacing rather than modifying. In other words, instead of running configuration management to apply a change on all of your servers, you build new servers and throw away the old ones.

Cloud environments greatly benefit from this method of creating change because the cost of deploying new VMs is often so low it is easier to do than to manage another system that can enforce configuration and keep instances running. Because servers are virtual (i.e., defined in software), you can apply the same practices used to build your applications as you can for building your server images.

> Traditional infrastructure with physical servers has an exact opposite optimization as the cloud when it comes to system creation.Provisioning a physical server takes a long time, and there is a great benefit in modifying existing operating systems rather than replace and throw away the old ones.

One of the problems with immutable infrastructure is building the chain of trust to create the golden image for deployment. Images, when built, should be verifiable(e.g., signing keys or image hash), and once deployed should not change.

Image creation also needs to be automated. One of the biggest issues with the historical use of golden images is they often relied on humans to run through checklists in time-consuming creation processes.

Tools exist (e.g., Packer by Hashicorp) to automate the build process, and there is no reason to have old images. The automation also allows the old checklist to become a script that can be audited and version-controlled. Knowing when the provisioning tools change and by whom is another aspect of immutable infrastructure that builds trust.

When a change occurs, and it should change frequently, you need a way to track what changed and why. Auditing will help identify what changed, and immutable infra‐structure can help you track back to a Git commit or pull request.

Having a tracked history also greatly helps with recovering from failure. If you have a VM image that is known to work and one that does not, you can deploy the previous, working version just as quickly as you deployed the new, broken version.

Immutable infrastructure is not a requirement of cloud native infrastructure, but the environment greatly benefits from this style of infrastructure management.

## Conclusion

Through the practices described in this chapter, you will be able to more easily control what runs in your infrastructure and track how it got there. Putting your policies in code will allow you to track changes, and you will not rely on humans to interpret the policy properly.

Auditing and immutable infrastructure give you much better information to make sure your systems are secure and help you recover from failure faster.

We will not discuss security in this book beyond the compliance testing requirements discussed earlier in this chapter, but you should keep up to date with security best practices for the technologies and cloud provider you use. One of the most important aspects of security is to apply “security in layers” in the entire stack.

In other words, just running an antivirus daemon on the host operating system is not enough to secure your applications. You’ll need to look at all layers of the infrastructure that you control and apply the security monitoring applicable to your needs.

All the testing described in this chapter should run in the same reconciler pattern described in Chapter 4. Gathering information to know the current state, looking for changes based on a set of rules, and then making the changes all fit cloud native patterns.

If you can implement your policies as code, you will go beyond technical benefits of the cloud and realize business benefits to cloud native patterns.

Historical and topographical auditing may not seem like clear benefits but will be very important as the infrastructure grows and the rate of change and application agility increases. Applying traditional methodologies to managing cloud infrastructure is not cloud native, and this chapter has shown you some of the benefits you should leverage and some of the challenges you will face.
