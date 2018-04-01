# Chapter 9 Implementing Cloud Native Infrastructure

If you thought cloud native infrastructure was a product you could buy or a cloud provider where you could run your servers, we are sorry to disappoint. It is not something you can benefit from without adopting the practices and changing how you build and maintain infrastructure.

It impacts much more than just servers, network, and storage. It is about how engineers manage applications just as much as it is about embracing failure.

A culture built around cloud native practices is very different from traditional technology and engineering organizations. We are not experts in addressing organizational cultures or structures, but if you’re looking to transform your organization, we suggest looking at values and lessons from implementing DevOps practices in high-performing organizations.

Some places to explore are Netflix’s culture deck, which promotes freedom and responsibility, and Amazon’s two-pizza teams, which promote autonomous groups with low overhead. Cloud native applications need to have the same decoupled characteristics as the teams that built them. Conway’s law describes it best: “organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations.”

As we close out this book, we’d like to focus on what areas are the most important as you adopt cloud native practices. We’ll also discuss some of the patterns in infrastructure that are predictors of change, so you know what to look for in the future.

## Where to Focus for Change

If you have existing infrastructure or traditional data centers, the transition to cloud native will not happen overnight. Trying to force a new way of managing infrastructure will likely fail if you have any sizable infrastructure footprint or more than two people managing it.

The main places of focus to adopt these patterns surprisingly have little to do with provisioning new servers or buying new software. To begin adopting cloud native infrastructure, it’s important that you focus on these areas first:

- People
- Architecture
- Chaos
- Applications

Don’t start changing your infrastructure until you have prepared these areas to run with the practices described in this book.

### People

> The ability to learn faster than your competitors may be the only sustainable competitive advantage.
>
> —Arie de Geus

As we discussed in Chapter 2, people are the hardest part of implementing any change. There are many reasons for their resistance, and it is the responsibility of those asking for a change to assist those it will affect.

Changes are easier when there is a driving incentive for the need to change. Being proactive to improve potential is a good motivator, but it can be hard to modify behavior without a sense of urgency.

Many reasons people are resistant to change come from fear. People like habits because they feel in control and avoid surprises.

In order to make a major transition in any technology successful, you need to work with people to minimize their fears. Give them a sense of ownership, and explain clear goals for the change. Make sure to highlight the similarities between the new and old technologies, especially when it comes to the role they will play after the change.

It is also important that people understand the change is not because of their failures with the old or existing systems. They need to understand that the requirements have changed and the environment is different, and you want them to be a part of the change because you respect what they have done and have confidence in what they can do.

They will need to learn new things, and for that to happen, failures are necessary, expected, and a sign of progress.

Encourage learning and experimentation, and reward people and systems that adapt to finding insights in data. One way you can do this by letting engineers explore new possibilities through freedoms such as “20 percent time.”

An agile system has no benefits if it doesn’t change. A system that does not adapt and improve will not be able to meet the needs of a business that is changing and learning.

Once you are able to excite people for change, you should empower them with trust and freedom. Continually guide them to align their goals with the business needs, and give them responsibility for what they were hired to manage.

If people are ready to adopt the practices in this book, there are few limitations in making it happen. For a much more in-depth guide to creating change in an organization, we recommend reading Leading Change by John P. Kotter (Harvard BusinessReview Press).

Changing the culture of your environment requires a lot of effort and buy-in from the organization, as does change the infrastructure where your applications run.The architecture you choose can have a significant impact on your ability to adopt cloud native patterns.

### Architecture

> Resilience, security, scalability, deployability, testability are architectural concerns.
>
> —Jez Humble

When migrating applications to cloud native infrastructure, you need to consider how the application is managed and designed. For example, 12-factor applications, the predecessor to cloud native applications, benefit from running on a platform.They are architected for minimal manual management, frequent changes, and resiliency. Many traditional applications are architected to resist automation, infrequent upgrades, and failures. You should consider the application’s architecture before moving it.

Individual application architecture is one concern, but you also need to consider how the application communicates with other services inside your infrastructure. Applications should communicate with protocols supported in cloud environments and through clearly defined interfaces. Keeping application scope small by adopting microservices can help keep inter-application interfaces defined and application deployment velocity up. However, adopting microservices will expose new problems, such as slower application communication and the need for distributed tracing and policy-controlled networking. Do not adopt microservices—and the problems they bring—without providing benefit in your infrastructure.

While you can fit just about any application to run in a container and be deployed with a container orchestrator, you will quickly doom your efforts if the first choice is to migrate all of your business-critical database servers.

It would be a better idea to first identify applications that are close to having the characteristics outlined in Chapter 1 and gain experience running them in a cloud native environment. Once you collectively gain experience and good practice around the simple applications, then you can decide what to move next.

Your servers and services are no different. Before you switch your infrastructure to be immutable, you should make sure it addresses your current problems and that you're aware of the new problems.

The most important architectural consideration in reliable systems is to strive for simple solutions. Software and systems naturally veer toward complexity, which will create instability. In a cloud, you release control of so many areas that it’s important to retain simplicity in the areas you can still control.

Whether you move your on-premises infrastructure to the cloud or create new solutions there, make sure you do the availability math in Chapter 1 and are prepared for the chaos.

### Chaos Management

> Embrace failure and expect chaos.
>
> —Andrew Spyker, Netflix

When you’re building cloud native applications and infrastructure, the goal is to create the minimal viable product (MVP) and iterate.5 Trying to dictate what your customers want or how they will use your product may be useful for a while, but successful applications will adapt instead of predict.

Customers’ needs change; applications need to change with them. You cannot plan and build a complete solution because, by the time the product is ready, the demands have changed.

Remaining agile and building on existing technology is important. Just as you import libraries for applications, you should consume IaaS and SaaS for infrastructure. The more you try to build yourself, the slower you will be able to provide value.

Whenever you release control of something, you risk the unexpected. If you’ve ever had an application break because an imported library was updated, you’ll know what this feels like.

Your application was dependent on functionality provided by the library, which changed. Should you remove the library and write your own functionality so you can control it? The answer is almost always no. Instead, you update your application to use the new library, or you bundle the functioning older version of the library with your application to temporarily avoid breakage.

The same is true for infrastructure running on a public cloud. You no longer have control of the hardwired network, what RAID controllers are used, or what version of the hypervisor runs your VMs. All you have are APIs that can provision abstractions on top of the underlying technology.

You do not control when the underlying technology is changed. If the cloud provider deprecates all large memory instance types, you have no choice but to comply. You either adapt to the new sizes, or you pay the cost (time and money) to change providers (see Appendix B about lock-in).

Most importantly, the fundamental infrastructure you build is no longer available from a single critical server. If you adopt cloud native infrastructure, whether you like it or not, you’re building a distributed system.

The old practices to keep services available by simply avoiding failure do not work.The goal is no longer the maximum number of nines you can engineer for—it is the minimum number of nines you can get away with.

Site Reliability Engineering explains it this way:

It’s both unrealistic and undesirable to insist that SLOs will be met 100% of the time: doing so can reduce the rate of innovation and deployment, require expensive, overly-conservative solutions, or both. Instead, it is better to allow an error budget—a rate at which the SLOs can be missed—and track that on a daily or weekly basis.

The goal of engineering is not available; it’s creating business value. You should make resilient systems but not at the expense of overengineering a solution to avoid chaos.

The old methods of testing changes to prevent downtimes won’t work either. It is nontrivial to create testing environments for large distributed systems. This is especially true when services are often updated and deployments are self-service.

It’s impossible to take a snapshot of an environment when there are 4,000 deploys per day at Netflix or 10,000 concurrently running versions of Facebook. Testing environments need to be dynamically segregated portions of production. The infrastructure needs to support this method of testing and embrace the failures that will come from frequently testing new code in production.

You can test for some chaos (see Chapter 5), but chaos is not predictable by definition. Prepare your infrastructure and applications to react to chaos predictably, and don’t try to avoid it.

### Applications

The purpose of infrastructure is to run applications. If your business were solely based on providing infrastructure to other companies, your success would still hinge on their ability to run applications.

If you build it, they are not guaranteed to come. Any abstractions you create need to improve the ability to run applications.

> **Avoid “Leaky Abstractions”**
>
> Abstractions do not completely hide the implementation details of what they are abstracting. The Law of Leaky Abstractions states that “All non-trivial abstractions, to some degree, are leaky.”
>
> What this means is the further you abstract something, the harder it is to hide the details of the thing.
>
> For example, application resource requests usually are abstracted by requesting a percentage of a CPU core, an amount of memory, and an amount of disk storage. The physical allocation of those resources is not managed by the application directly (theAPI provisions them), but the request for the resources makes it obvious that the systems that run the application have those types of resources available.
>
> If instead, the abstraction were a percentage of a server or data center (e.g., 50 servers,0.2 data centers), that abstraction does not have the same meaning because there is no such thing as a single size server or data center unit. Make sure the abstractions created are meaningful for the applications that will use them.

The benefit of cloud native practices is that as you improve your ability to run applications, you also improve your ability to run infrastructure. If you follow the patterns Chapters 3–6, you will be well adapted to use applications to continually improve and adapt your infrastructure to the applications’ needs.

Focus on building applications that are small in scope, easy to adapt, easy to operate, and resilient when failure happens. Make sure those applications are responsible for all infrastructure management and changes. If you do that, you will have created cloud native infrastructure.

## Predicting the Future

If you have already adopted the patterns and practices in this book, you are now in uncharted territory. The companies that run the largest infrastructure in the world have adopted these practices. Whatever new patterns they have for running infrastructure have not been publicly disclosed or are still being discovered.

The good news is your infrastructure is now designed to be agile and change. You can more easily adapt to whatever new challenges you have.

To look at the road ahead, instead of looking at existing infrastructure patterns, we can look at where infrastructure gets its inspiration—software. Almost every pattern that has been adopted by infrastructure came from patterns in software development.

For an example, look at distributed applications. Many years ago, when software was exposed to the internet, running the application on a single server under a single codebase would not scale. This included performance and process limitations for managing the applications.

The application needed to be duplicated onto additional servers and then load balanced to meet demand. When this reached limits, it was broken apart into smaller components, and APIs were created to remotely call the functionality over HTTP instead of through an application library.

Infrastructure has taken a similar approach to scaling; it just adapts slower than software in most areas. Modern infrastructure platforms such as Kubernetes break apart rolls of infrastructure management into smaller components. The components can scale independently depending on their performance bottleneck (CPU, memory, and I/O) and can be iterated on quickly.

Some applications do not have the same scaling requirements and do not need to adapt to a new architecture. One of the roles of an engineer is to know when, and when not, to adopt new technologies. Only those who know the limitations and bottlenecks of their stack should decide what is the right direction.

Keep an eye on what new solutions applications develop as they find new limitations.If you can understand what the limitation is and why the change was made, you will always be ahead of the curve in infrastructure.

## Conclusion

Our goal with this book was to help you better understand what cloud native infrastructure is and why you would want to adopt it. While we believe it has many benefits over traditional infrastructure, we do not want you to blindly use any technology without understanding it.

Do not expect to gain all of the benefits without adopting the culture and processes that come with it. Just running infrastructure in a public cloud or running a container orchestrator will not change the process for how that infrastructure adapts.

It is easier to manually create a handful of VMs in Amazon Web Services, SSH into each one, and create a Kubernetes cluster than it is to change how people work. The former is not cloud native.

Remember, the applications that provision your infrastructure are not static snapshots in time; they should continually run and drive the infrastructure toward the desired state. Managing infrastructure is not about maintaining hosts. You need to create abstractions for resources, APIs to represent those abstractions, and applications to consume them.

The goal is to meet the needs of your applications and to write applications that are the software equivalent of your business process and job functions. The software needs to easily adapt, as processes and functions change frequently.

When you master that, you will continually find challenges in creating new abstractions, keeping services resilient, and pushing the limits of scalability. If you are able to find new limitations and useful abstractions, please give back to the communities you are a part of.

Open source and giving back through research and community is how these patterns have emerged from the environments that pioneered them. Sharing allows more innovation and insights to spread and makes the longevity and adaptability of these practices easier for everyone.

Innovation doesn’t come only from finding solutions or building the next great product. It takes the seemingly unimportant form of asking questions, speaking up, and failing.

Please keep doing all those things, especially failing and sharing.