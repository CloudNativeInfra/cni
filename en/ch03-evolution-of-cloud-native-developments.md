# Chapter 3 Evolution of Cloud Native Deployments

We discussed in the previous chapter what the requirements are before you adopt cloud native infrastructure. Having API-driven infrastructure provisioning (IaaS) will be a requirement before you deploy.

In this chapter, we explore taking the idea of cloud native infrastructure topologies and realizing them in a cloud. We’ll learn common tools and patterns that help operators control their infrastructure.

The first step in deploying infrastructure is being able to represent it. Traditionally this could have been handled on a whiteboard, or, if you’re lucky, in documentation stored on a company wiki. Today, things are moving into a more programmatic space, and infrastructure representation is commonly documented in ways that make it easy for applications to interpret it. Regardless of the avenue in which it is represented, the need to comprehensively represent the infrastructure remains the same.

As one would expect, refining infrastructure in the cloud can range from very simple designs to very complex designs. Regardless of the complexity, great attention to detail must be given to infrastructure representation to ensure the design is reproducible. Being able to clearly transmit ideas is even more important. Therefore, having a clear, accurate, and easy-to-understand representation of infrastructure-level resources is imperative.

We also gain a lot from having a well-crafted representation:

- Infrastructure design can be shared and versioned over time.
- Infrastructure design can be forked and altered for unique situations.
- The representation implicitly is documentation.

As we move forward in the chapter, we will see how infrastructure representation is the first step in infrastructure deployment. We will explore both the power and potential pitfalls of representing infrastructure in different ways.

## Representing Infrastructure

To begin, we need to understand the two roles in representing infrastructure: the author and the audience.

The author is what will be defining the infrastructure, usually a human operator or administrator. The audience is what will be responsible for interpreting the infrastructure representation. Sometimes this is a human operator performing manual steps, but hopefully, it is a deployment tool that can automatically parse and create the infrastructure. The better the author is at accurately representing the infrastructure, the more confidence we can gain in the audience’s ability to interpret the representation.

The main concern while authoring infrastructure representation is to make it understandable for the audience. If the target audience is humans, the representation might come in the form of a technical diagram or abstracted code. If the target audience is a program, the representation may need more detailed information and concrete implementation steps.

Despite the audience, the author should make it easy for the audience to consume.This becomes difficult as complexity increases and if the representation is consumed by both humans and programs.

Representation needs to be easy to understand so that it can be accurately parsed.Easy-to-read but inaccurately parsed representation negates the entire effort. The audience should always strive to interpret the representation they were given, and never make assumptions.

In order to make the representation successful, the interpretation needs to be predictable. The best audiences are ones that will fail quickly and obviously if the author neglected an important detail. Having predictability will reduce mistakes and errors when changes are applied and will help build trust between authors and audiences.

### Infrastructure as a Diagram

We all have walked up to a whiteboard and began drawing an infrastructure diagram.Usually, this starts off with a cloud shape in the corner that represents the internet, and some arrows pointing to boxes. Each of these boxes represents a component of the system, and the arrows represent the interactions between them. Figure 3-1 is an example of an infrastructure diagram.

![f-3-1](../images/f-3-1.jpg)

Figure 3-1. Simple infrastructure diagram

This is a wonderfully effective approach to brainstorming and transmitting ideas to other humans. It allows for a quick and powerful representation of complex infrastructure designs.

Pictures work well for humans, large crowds, and CEOs. These diagrams also work well because they use common idioms to represent relationships. For example, this box may send data to that box, but will never send data to this other box.

Unfortunately, diagrams are virtually impossible for a computer to understand. Until computer vision catches up, infrastructure diagrams remain a representation to be interpreted by eyeballs, not code.

> **Deploying from a Diagram**
>
> In Example 3-1 we look at a familiar snippet of code from a bash_history file. It represents an infrastructure operator working as the audience for a diagram describing abasic server with networking, storage, and sublet service running.
>
> The operator has manually deployed a new virtual machine and has SSHed into the machine to begin provisioning it. In this case, the human acts as the interpreter for the diagram and then take action in the infrastructure environment.
>
> Most infrastructure engineers have done this in their career, and these steps should be alarmingly familiar to some system administrators.

Example 3-1. bash_history

```bash
sudo emacs /etc/networking/interfaces
sudo ifdown eth0
sudo ifup eth0
sudo fdisk -l
sudo emacs /etc/fstab
sudo mount -a
sudo systemctl enable kubelet
```

### Infrastructure as a Script

As a systems administrator, part of your job is to make changes across complex systems; it is also your responsibility to ensure that those changes are correct. The need to have these changes spread across vast systems is very real. Unfortunately, so is human error. It’s no surprise that administrators write convenience scripts for the job.

Scripts can help reduce the amount of human error on repeated tasks, but automation is a double-edged sword. It does not imply accuracy or success.

> For SRE, automation is a force multiplier, not a panacea. Of course, just multiplying force does not naturally change the accuracy of where that force is applied: doing automation thoughtlessly can create as many problems as it solves.
>
> —Niall Murphy with John Looney and Michael Kacirek, The Evolution ofAutomation at Google

Writing scripts is a good way to automate steps to produce the desired outcome. The script could perform miscellaneous tasks, such as installing an HTTP server, configuring it, and running it. However, the steps in the scripts rarely take into consideration their outcome or the state of the system when they’re invoked.

The script, in this case, is the encoded data that represents what should happen to create the desired infrastructure. Another operator or administrator could evaluate your script and hope to understand what the script is doing. In other words, they’d be interpreting your infrastructure representation. Understanding the desired infra‐structure relies on understanding how the steps affect the system.

The runtime for your script would execute the steps in the order they are defined, but the runtime has no knowledge of what it is producing. The script is the code, and the outcome of the script is, hopefully, the desired infrastructure.

This works well for very basic scenarios, but there are some flaws in this approach.The most obvious flaw would be running the same script and getting two outcomes.

What if the environment where the script was first to run is drastically different than the environment where it is run a second time? Scientifically speaking, this would be analogous to a flaw in a procedure and would invalidate the data from the experiment.

Another flaw with using scripts to represent infrastructure is the lack of declarative state. The runtime for the script doesn’t understand the end state because it is only provided steps to perform. Humans need to interpret the desired outcome from the steps to understand how to make changes.

We have all seen code that is hard to understand as a human. As the complexity of a provisioning script grows, our ability to interpret what the script does diminishes.Furthermore, as your infrastructure needs change over time, the script will inevitably need to change.

Without abstracting steps into a declarative state, the script will grow to try to create procedures for every possible initial state. This includes abstracting away steps and differences between operating systems (e.g., apt and DNF) as well as verifying what steps can safely be skipped.

Infrastructure as code brought about tools that provided some of these abstractions to help reduce the burden and maintenance of managing infrastructure as scripts.

> **Deploying from a Script**
>
> The next evolution of creating infrastructure is to begin to take the previous process of manually managing infrastructure and simplify it for the administrator by encapsulating the work in a script. Imagine we had a bash script called createVm.sh that would create a virtual machine from our local workstation.
>
> The script would take two arguments. The first would be the static IP address to assign to a network interface on the virtual machine. The second would be a size in gigabytes that would be used to create a volume and attach it to the virtual machine.
>
> Example 3-2 shows a very basic representation of infrastructure as a script. The script will provide the new infrastructure and run an arbitrary provision script on the newly created infrastructure. This script could be evolved to be highly customizable and could (dangerously) be automated to run at the click of a button.

Example 3-2. Infrastructure as a script

```bash
#!/bin/bash

# Create a VM with a NIC on 10.0.0.17 and a 100gb volume

createVm.sh 10.0.0.17 100

# Transfer the bootstrapping script

scp ~/vm_provision.sh user@10.0.0.17:vm_provision.sh -v

# Run the bootstrap script

ssh user@10.0.0.17 sh ~/vm_provision.sh
```

### Infrastructure as Code

Configuration management was once the king of representing infrastructure.1 We can think of configuration management as abstracted scripts that automatically consider the initial state to perform the proper procedures. Most importantly, configuration management allows authors to declare the desired state of a node instead of every step needed to achieve it.

Configuration management was the first step in the direction of infrastructure as code, but the tooling has rarely reached beyond the scope of a single server. Configuration management tools do an excellent job of defining specific resources and what their state should be, but complications arise as infrastructure requires coordination between resources.

For example, the DNS entry for a service should not be available until the service has been provisioned. The service should not be provisioned before the host is available.Failure to coordinate multiple resources across independent nodes has made the abstractions provided by configuration management inadequate. Some tools have added the ability to coordinate configuration between resources, but the coordination has often been procedural, and the responsibility has fallen on humans to order resources and understand desired state.2

Your infrastructure does not consist of independent entities without communication.Your tools to represent the infrastructure need to take that into consideration. So another representation was needed to manage low-level abstractions (e.g., operating systems), as well as provisioning and coordination.

In July of 2014, an open source tool that embraced the idea of a higher level of abstraction for infrastructure as the code was released. The tool, called Terraform, has been fantastically successful. It was released at the perfect time when configuration management was well established and public cloud adoption was on the rise. Users began to see the limitations of the tools for the new environment, and Terraform was ready to address their needs.

> We originally thought of infrastructure as code in 2011. We noticed we were writing tools to solve the infrastructure problem for many projects, and wanted to standardize the process.
>
> —Mitchell Hashimoto, CEO of Hashicorp and creator of Terraform

Terraform represents infrastructure using a specialized domain-specific language(DSL), which provides a good compromise between human-understandable images and machine-parsable code. Some of the most successful parts of Terraform are its abstracted view of infrastructure, resource coordination, and ability to leverage existing tools when applicable. Terraform talks to cloud APIs to provision infrastructure and can use configuration management to provision nodes when necessary.

This was a fundamental shift in the industry, as we saw one-off provisioning scripts fade into the background. More and more operators began developing infrastructure representation in the new DSL. Engineers who used to manually operate on infrastructure were now developing code.

The new DSL solved the problems of representing infrastructure as a script and became a standard for representing infrastructure. Engineers found themselves developing a better representation of infrastructure as code and allowing Terraform to interpret it. Just as with configuration management code, engineers began storing their infrastructure representations in version control and treating infrastructure architecture as they treat software.

By having a standardized way of representing infrastructure, we liberated ourselves from the pain of having to learn every proprietary cloud API. While not all cloud resources could be abstracted in a single representation, most users could accept cloud lock-in in their code.3 Having a human-readable and machine-parsable representation of infrastructure architecture, not just independent resource declaration, changed the industry forever.

> **Deploying from Code**
>
> After running into the challenges of deploying infrastructure as a script, we have created a program that will parse input and take action against infrastructure for us.
>
> Example 3-3 shows a Terraform configuration taken from the Terraform open source repository. Notice how the code has variables and will need to be parsed at runtime.
>
> Declarative representation of infrastructure is important because it doesn’t define individual steps to create the infrastructure. This allows us to separate what needs to be provisioned from how it gets provisioned. This is what makes this infrastructure representation a new paradigm; it was also a first step in the evolution toward infrastructure as software.
>
> Representing infrastructure in this way is a powerful, common practice for engineers.The user could use Terraform to terraform apply the infrastructure.

Example 3-3. example.tf

```bash
# Create our DNSimple record
resource "dnsimple_record" "web" {
domain = "${var.dnsimple_domain}"
name = "terraform"
value = "${hostname}"
type = "CNAME"
ttl = 3600
}
```

### Infrastructure as Software

Infrastructure as the code was a powerful move in the right direction. But the code is a static representation of infrastructure and has limitations. You can automate the process of deploying code changes, but unless the deployment tool runs continually, there will still be configuration drift. Deployment tooling traditionally works only in a single direction: it can only create new objects, and can’t easily delete or modify existing objects.

To master infrastructure, our deployment tools need to work from the initial representation of infrastructure and mutate the data to make more agile systems. As we begin to look at our infrastructure representation as a versionable body of data that continually enforces the desired state, we need the next step of infrastructure as software.

Terraform took lessons from configuration management and improved on that concept to better provision infrastructure and coordinate resources. Applications need an abstraction layer to utilize resources more efficiently. As we explained in Chapter1, applications cannot run directly on IaaS and need to run on a platform that can manage resources and run applications.

IaaS presented raw components as provisional API endpoints and platforms present APIs for resources that are more easily consumed by applications. Some of those resources may provision IaaS components (e.g., load balancers or disk volumes), but many of them will be being managed by the platform (e.g., compute resources).

Platforms expose a new layer of infrastructure and continually enforce the desired state. The components of the platforms are also applications themselves that can be managed with the same desired state declarations.

The API machinery allows users to reap the benefits of standardizing infrastructure as code and adds the ability to version and change the representation over time. APIs allow a new way of consuming resources through standard practices such as API versioning. Consumers of the API can build their applications to a specific version and trust that their usage will not break until they choose to consume a new API version.Some of these practices are critical features missing from the previous infrastructure as code tools.

With software that continually enforces representation, we can now guarantee the current state of our systems. The platform layer becomes much more consumable for applications by providing the right abstractions.

You may be drawing parallels between the evolution of infrastructure and the evolution of software. The two layers in the stack have evolved in remarkably similar ways.

> Software is eating the world.
>
> —Marc Andreessen

Encapsulating infrastructure and thinking of it as a versioned API is remarkably powerful. This dramatically increases the velocity of a software project responsible for interpreting a representation. Abstractions provided by a platform are necessary to keep up with the quickly growing cloud. This new pattern is the pattern of today and the one that has been proven to scale to unfathomable numbers for both infrastructure and applications.

> **Deploying from Software**
>
> A fundamental difference between infrastructure as code and infrastructure as software is that software has the ability to mutate the data store, and thus the representation of infrastructure. It is up to the software to manage the infrastructure, and the representation is a give-and-take between the operator and the software.
>
> In Example 3-4 we take a look at a YAML representation of infrastructure. We can trust the software to interpret this representation and negotiate the outcome of the YAML for us.
>
> We start with a representation of the infrastructure just as we did when developing infrastructure as code. But in this example, the software will run continually and ensure this representation over time. In a sense, this is still read-only, but the software can extend this definition to add its own meta information, such as tagging and resource creation times.

Example 3-4. infrastructure.yaml

```yaml
  location: "New York 1"
  name: example
  dns:
      fqdn: infra.example.com
  network:
    cidr: 172.0.0.0/12
  serverPools:
    - bootstrapScript: /home/user/bootstrap.sh
      diskSize: 40gb
      firewalls:
        - rules:
            - ingressFromPort: 443
              ingressProtocol: tcp
              ingressSource: 0.0.0.0/0
              ingressToPort: 443
      maxCount: 1
      minCount: 1
      image: centos-amd64-7
      subnets:
        - cidr: 172.0.100.0/24
```

## Deployment Tools

We now understand the two roles in deploying infrastructure:

**The author**

The component defining the infrastructure

**The audience**

The deployment tool interpreting the representation and taking action

There are many avenues in which we might represent infrastructure, and the component taking action is a logical reflection of the initial representation. It’s important to accurately represent the proper layer of the infrastructure and to eliminate complexity in that layer as much as possible. With simple, targeted releases, we will be able to more accurately apply the changes needed.

Site Reliability Engineering (O’Reilly, 2016) concludes that “simple releases are generally better than complicated releases. It is much easier to measure and understand the impact of a single change rather than a batch of changes released simultaneously.”

As our representation of infrastructure has changed over time to be more abstracted from underlying components, our deployment tools have changed to match the new abstraction targets.

We are approaching the infrastructure as software boundary and can notice the early signs of a new era of infrastructure deployment tools. Open source projects across the internet are popping up that claim to be able to manage infrastructure over time. It is the engineer’s job to know what layer of infrastructure that project manages and how it impacts their existing tools and other infrastructure layers.

The first step in the direction of cloud native infrastructure was taking provisioning scripts and scheduling them to run continually. Some engineers would intentionally engineer these scripts to work well with being scheduled over time. We began to see elaborate global locking mechanisms, advanced scheduling tactics, and distributed scheduling approaches.

This was essentially what configuration management promised, albeit at a more resource-specific abstraction. Thanks to the cloud, the days of automated scripts to manage infrastructure have come and gone.

> Automation is dead.
>
> —Charity Majors, CEO of Honeycomb

We are imagining a world where we begin to look at infrastructure tooling completely differently. If your infrastructure is designed to run in the cloud, then IaaS is not the problem you should be solving. Consume the provided APIs, and build the new layer of infrastructure that can be directly consumed by applications.

We are in a special place in the evolution of infrastructure where we take the leap into designing infrastructure deployment tools as elegant applications from day one.

Good deployment tools are tools that can quickly go from a human-friendly representation of infrastructure to working infrastructure. Better deployment tools are tools that will undo any changes that have been made that disagree with the initial representation. The best deployment tools do all of this without needing human involvement.

As we build these applications, we cannot forget important lessons learned from pro‐visioning tools and software practices that are crucial for handling complex systems.

Some key aspects of deployment tools we’ll look at are idempotency and handling failure.

### Idempotency

The software can be idempotent, meaning you must be able to continually feed it the same input, and always get the same output.

In technology, this idea was made famous by the Hypertext Transfer Protocol(HTTP) via idempotent methods like PUT and DELETE. This is a very powerful idea, and advertising the guarantee of idempotency in software can drastically shape complex software applications.

One of the lessons learned in early configuration management tools was idempotency. We need to remember the value this feature offered infrastructure engineers, and continue to build this paradigm into our tooling.

Being able to automatically create, update, or delete infrastructure with the guarantee that no matter how often you run the task it will always output the same is quite exciting. It allows for operators to begin to work on automating tasks and chores.What used to be a sizable amount of work for an operator can now be as simple as a button click in a web page.

The idempotent guarantee is also effective at helping operators perform quality science on their infrastructure. Operators could begin replicating infrastructure in many physical locations, with the knowledge that someone else repeating their procedure would get the same thing.

We began to notice entire frameworks and toolchains built around this idea of automating arbitrary tasks for repeatability.

As it was with software, so it became with infrastructure. Operators began automat‐ing entire pipelines of managing infrastructure using these representations and deployment tools. The work of an operator now became developing the tooling around automating these tasks and no longer performing the tasks themselves.

### Handling Failure

Any software engineer can tell you about the importance of handling failures and edge cases in their code. We naturally began to develop these same concerns as infra‐structure administrators.

What happens if a deployment job fails in the middle of its execution, or, more importantly, what should happen in that case?

Designing our deployment tools with failure in mind was another step in the right direction. We found ourselves sending messages or registering alerts in a monitoring system. We kept detailed logs of the automation tasks. We even made the leap to chaining logic together in the case of failure.

We obsessed over failures. We began taking action in the case of failures and took action when they happened.

But building systems around the idea that a single component might fail is drastically different than building components to be more resilient to failure. Having a component retry or adjust its approach based on failure is taking the resiliency of a system a step deeper into the software. This allows for a much stabler system and reduces the overall support needed from the system itself.

Design components for failure, not systems.

**Eventual consistency**

In the name of designing components for failure, we need to learn a term that describes a common approach to handling failure.

Eventual consistency means to attempt to reconcile a system over time. Larger systems and smaller components can both adhere to this philosophy of retrying a failed process over time.

One of the benefits of an eventually consistent system is that an operator can have confidence that it will ultimately reach the desired state. One of the concerns with these systems is that sometimes they might take inappropriately long amounts of time to reach the desired state.

Knowing when to choose a stable but slow system versus an unreliable but fast system is a technical decision an administrator must make. The important relationship to note in this decision is that systems exchange speed for reliability. It’s not always easy, but if in doubt, always choose reliable systems.

**Atomicity**

Contrary to eventually consistent systems is the atomic system, a guaranteed transaction that dictates that the job succeeded in its entirety. If the job cannot complete, it will revert any changes it made and fail completely.

Imagine a job that needed to create 10 virtual machines. The job gets to the 7th virtual machine and something goes wrong. According to the eventual consistency approach, we would just try the job over and over in the hopes of eventually getting10 virtual machines.

It’s important to look at the reason we only were able to create seven virtual machines.Imagine there was a limit on how many virtual machines a cloud would let us create.The eventual consistency model would continue to try to create three more machines and inevitably fail every time.

If the job were engineered to be atomic, it would hit the limit on the seventh machine and realize there was a catastrophic failure. The job would then be responsible for deleting the partial system.

So the operator can rest assured that they either get the system as intended in its entirety or nothing at all. This is a powerful idea, as many components in infrastructure are dependent on the rest of the system to be in place in order for it to work.

We can introduce confidence in exchange for the inconvenience. That is to say, the administrator would be confident the state of their system would never change unless a perfect change could be applied. In exchange for this perfect system, the operator might face great inconvenience, as the system could require a lot of work to keep running smoothly.

Choosing an atomic system is safe, but potentially not what we want. The engineer needs to know what system they want and when to pick atomicity versus eventual consistency.

## Conclusion

The pattern of deploying infrastructure is simple and has remained unchanged since before the cloud was available. We represent infrastructure, and then using some device, we manifest the infrastructure into reality.

The infrastructure layers of the stack have astonishingly similar histories to the software application layers. Cloud native infrastructure is no different. We begin to find ourselves repeating history, and learning age-old lessons in new guises.

What is to be said about the ability to predict the future of the infrastructure industry if we already know the future of its software counterpart?

Cloud native infrastructure is a natural, and possibly expected, the evolution of infrastructure. Being able to deploy, represent, and manage it in reliable and repeatable ways is a necessity. Being able to evolve our deployment tools over time, and shift our paradigms of how this is done, is critical to keeping our infrastructure in a space that can keep up with supporting its application-layer counterpart.