# Chapter 4 Designing Infrastructure Applications

In the previous chapter, we learned about representing infrastructure and the various approaches and concerns with deployment tools around it. In this chapter, we look at what, it takes to design applications that deploy and manage infrastructure. We heed the concerns of the previous chapter and focus on opening up the world of infrastructure as software, sometimes called infrastructure as an application.

In a cloud native environment, traditional infrastructure operators need to be infra‐structure software engineers. It is still an emerging practice and differs from other operational roles in the past. We desperately need to begin exploring patterns and setting standards.

A fundamental difference between infrastructure as code and infrastructure as software is that software continually runs and will create or mutate infrastructure based on the reconciler pattern, which we will explain later in this chapter. Furthermore, the new paradigm behind infrastructure as software is that the software now has a more traditional relationship with the data store and exposes an API for defining the desired state. For instance, the software might mutate the representation of infrastructure as needed in the data store, and very well could manage the data store itself! Desired state changes to reconcile are sent to the software via the API instead of static code repo.

The first step in the direction of infrastructure as software is for infrastructure operators to realize they are software engineers. We welcome you all warmly to the field!Previous tools (e.g., configuration management) had similar goals to change infra‐structure operators’ job function, but often the operators only learned how to write a limited DSL with narrow scope application (i.e., single node abstraction).

As an infrastructure engineer, you are tasked not only with having a mastery of the underlying principals of designing, managing, and operating infrastructure, but also with taking your expertise and encapsulating it in the form of a rock-solid application. These applications represent the infrastructure that we will be managing and mutating.

Engineering software to manage infrastructure is not an easy undertaking. We have all the major problems and concerns of a traditional application, and we are developing in an awkward space. It’s awkward in the sense that infrastructure engineering is an almost ridiculous task of building software to deploy infrastructure so that you can then run the same software on top of the newly created infrastructure.

To begin, we need to understand the nuances of engineering software in this new space. We will look at patterns proven in the cloud native community to understand the importance of writing clean and logical code in our applications. But first, where does infrastructure come from?

## The Bootstrapping Problem

On Sunday, March 22, 1987, Richard M. Stallman sent an email to the GCC mailing list to report successfully compiling the C compiler with itself:

This compiler compiles itself correctly on the 68020 and did so recently on the vax. It recently compiled Emacs correctly on the 68020 and has also compiled tex-in-C and Kyoto Common Lisp. However, it probably still has numerous bugs that I hope you will find for me.

I will be away for a month, so bugs reported now will not be handled until then.—Richard M. Stallman

This was a critical turning point in the history of software, as we were engineering software to bootstrap itself. Stallman had literally created a compiler that could compile itself. Even accepting this statement as truth can be philosophically difficult.

Today we are solving the same problem with infrastructure. Engineers must come up with solutions to almost impossible problems of a system bootstrapping itself and coming to life at runtime.

One approach is to provide the first bit of infrastructure in the cloud and infrastructure applications manually. While this approach does work, it usually comes with the caveat that the operator should destroy the initial bootstrap infrastructure after more appropriate infrastructure has been deployed. This approach is tedious, difficult to repeat, and prone to human errors.

A more elegant and cloud native approach to solving this problem is to make the(usually correct) assumption that whoever is attempting to bootstrap infrastructure software has a local machine that we can use to our advantage. The existing machine(your computer) serves as the first deployment tool, to create infrastructure in a cloud automatically. After the infrastructure is in place, your local deployment tool can then deploy itself to the newly created infrastructure and continually run. Good deployment tools will allow you to easily clean this up when you are done.

After the initial infrastructure bootstrap problem is solved, we can then use the infrastructure applications to bootstrap new infrastructure. The local computer is now taken out of the equation, and we are running entirely cloud native at this point.

## The API

In earlier chapters, we discussed the various methods for representing infrastructure.In this chapter, we will be exploring the concept of having an API for infrastructure.

When the API is implemented in software, it more than likely will be done via a data structure. So, depending on the programming language you are using, it’s safe to think of the API as a class, dictionary, array, object, or struct.

The API will be an arbitrary definition of data values, maybe a handful of strings, a few integers, and a boolean. The API will be encoded and decoded from some sort of encoding standing like JSON or YAML, or might even be stored in a database.

Having a versionable API for a program is a common practice for most software engineers. This allows the program to move, change, and grow over time. Engineers can advertise to support older API versions and offer backward-compatibility guarantees. In engineering infrastructure as software, using an API is preferred for these reasons.

Finding an API as the interface for infrastructure is one of the many clues that a user will be working with infrastructure as software. Traditionally, infrastructure as code is a direct representation of the infrastructure a user will be managing, whereas an API might be an abstraction on top of the exact underlying resources being managed.1

Ultimately, an API is just a data structure that represents infrastructure.

## The State of the World

Within the context of an infrastructure as software tool, the world is the infrastructure that we will be managing. Thus, the state of the world is just an audited representation of the world as it exists to our program.

The state of the world will ultimately make its way back to an in-memory representation of the infrastructure. These in-memory representations should map to the original API used to declare infrastructure. The audited API, or state of the world, typically will be needed to be saved.

A storage medium (sometimes referred to as a state store) can be used to store the freshly audited API. The medium can be any traditional storage system, such as a local filesystem, cloud object storage, or a database. If the data is stored in a filesystem-like store, the tool will most likely encode the data in a logical way so that the data can easily be encoded and decoded at runtime. Common encodings for this include JSON, YAML, and TOML.

As you begin to engineer your program, you might catch yourself wanting to store privileged information with the rest of the data you are storing. This may or may not be best practice, depending on your security requirements and where you plan on storing data.

It is important to remember that storing secrets can be a vulnerability. While you are designing software to control the most fundamental part of the stack, security is critical. So it’s usually worth the extra effort to ensure secrets are safe.

Aside from storing meta information about the program and cloud provider credentials, an engineer will also need to store information about infrastructure. It is important to remember that the infrastructure will be represented in some way, ideally, one that's easy for the program to decode. It is also important to remember that making changes to a system does not happen instantly, but rather over time.

Having these pieces of data stored and easily accessible is a large part of designing the infrastructure management application. The infrastructure definition alone is quite possibly the most intellectually valuable part of the system. Let’s take a look at a basic example to see how this data and the program will work together.

It is important to review Examples 4-1 through 4-4, as they are used as concrete examples for lessons further in the chapter.

A filesystem state store example

Imagine a data store that was simply a directory called state. Within the directory, there would be three files:

- meta_information.yaml
- secrets.yaml
- infrastructure.yaml

This simple data store can accurately encapsulate the information needed to be preserved in order to effectively manage infrastructure.

The secrets.yaml and infrastructure.yaml files store the representation of the infrastructure, and the meta_information.yaml file (Example 4-1) stores other important information such as when the infrastructure was last provisioned, who provisioned it, and logging information.

Example 4-1. state/meta_information.yaml

```yaml
lastExecution:
  exitCode: 0
  timestamp: 2017-08-01 15:32:11 +00:00
  user: kris
  logFile: /var/log/infra.log
```

The second file, secrets.yaml holds private information, used to authenticate in arbitrary ways throughout the execution of the program (Example 4-2).

Again, storing secrets in this way might be unsafe. We are using secrets.yaml merely as an example.

Example 4-2. state/secrets.yaml

```yaml
apiAccessToken: a8233fc28d09a9c27b2e2f
apiSecret: 8a2976744f239eaa9287f83b23309023d
privateKeyPath: ~/.ssh/id_rsa
```

The third file, infrastructure.yaml, would contain an encoded representation of the API, including the API version used (Example 4-3). Here can we find infrastructure representation, such as for a network and DNS information, firewall rules, and virtual machine definitions.

Example 4-3. state/infrastructure.yaml

```yaml
location: "San Francisco 2"
name: infra1
dns:
    fqdn: infra.example.com
network:
  cidr: 10.0.0.0/12
  serverPools:
  - bootstrapScript: /opt/infra/bootstrap.sh
    diskSize: large
    workload: medium
    memory: medium
    subnetHostsCount: 256
    firewalls:
      - rules:
          - ingressFromPort: 22
            ingressProtocol: tcp
            ingressSource: 0.0.0.0/0
            ingressToPort: 22
    image: ubuntu-16-04-x64
```

The infrastructure.yaml file at first might appear to be nothing more than an example of infrastructure as code. But if you look closely, you will see that many of the directives defined are an abstraction on top of the concrete infrastructure. For instance, the subnetHostsCount directive is an integer value and defines the intended number of hosts for a subnet. The program will manage to section off the larger classless interdomain routing (CIDR) value defined in a network for the operator. The operator does not declare a subnet, just how many hosts they would like. The software reasons for the rest of the operator.

As the program runs, it might update the API and write the new representation out to the data store (which in this case is simply a file). To continue with our subnetHostsCount example, let’s say that the program did pick out a subnet CIDR for us. The new data structure might look something like Example 4-4.

```yaml
location: "San Francisco 2"
name: infra1
dns:
    fqdn: infra.example.com
network:
  cidr: 10.0.0.0/12
serverPools:
  - bootstrapScript: /opt/infra/bootstrap.sh
    diskSize: large
    workload: medium
    memory: medium
    subnetHostsCount: 256
    assignedSubnetCIDR: 10.0.100.0/24
    firewalls:
      - rules:
          - ingressFromPort: 22
            ingressProtocol: tcp
            ingressSource: 0.0.0.0/0
            ingressToPort: 22
    image: ubuntu-16-04-x64
```

Notice how the program wrote the assignedSubnetCIDR directive, not the operator.Also, remember how the program updating the API is a sign that a user is interacting with infrastructure as software.

Now, remember this is just an example and does not necessarily advocate for using an abstraction for calculating a subnet CIDR. Different use cases may require different abstractions and implementation in the application. One of the beautiful and powerful things about building infrastructure applications is that users can engineer the software in any way they find necessary to solve their set of problems.

The data store (the infrastructure.yaml file) can now be thought of as a traditional data store in the software engineering realm. That is, the program can have full write control over the file.

This introduces risk, but also a great win for the engineer, as we will discover. The infrastructure representation doesn’t have to be stored in files on a filesystem. Instead, it can be stored in any data storage such as a traditional database or key/value storage system.

To understand the complexities of how software will handle this new representation of infrastructure, we have to understand the two states in the system—the expected state in the form of the API, which is found in the infrastructure.yaml file, and the actual state that can be observed in reality (or audited), or the state of the world.

In this example, the software hasn’t done anything or taken any action yet, and we are at the beginning of the management timeline. Thus, the actual state of the world would be nothing, while the expected state of the world would be whatever is encapsulated in the infrastructure.yaml file.

## The Reconciler Pattern

The reconciler pattern is a software pattern that can be used or expanded upon for managing cloud native infrastructure. The pattern enforces the idea of having two representations of the infrastructure—the first being the actual state of the infrastructure, and the second being the expected state of the infrastructure.

The reconciler pattern will force the engineer to have two independent avenues forgetting either of these representations, as well as to implement a solution to reconcile the actual state into the expected state.

The reconciler pattern can be thought of as a set of four methods, and four philosophical rules:

1. Use a data structure for all inputs and outputs.
2. Ensure that the data structure is immutable.
3. Keep the resource map simple.
4. Make the actual state match the expected state.

These are powerful guarantees that a consumer of the pattern can rely on. Furthermore, they liberate the consumer from the implementation details.

### Rule 1: Use a Data Structure for All Inputs and Outputs

The methods implementing the reconciler pattern must only accept and return a data structure.2 The structure must be defined outside the context of the reconciler implementation, but the implementation must be aware of it.

By only accepting a data structure for input and returning one as output, the consumer can reconcile any structure defined in their data store without having to be bothered with how that reconciliation takes place. This also allows the implementations to be changed, modified, or switched at runtime or with different versions of the program.

While we want to adhere to the first rule as often as possible, it’s also very important to never tightly couple a data structure and codebase. Always observe best abstraction and separation practices, and never use subsets of the API to pass to/from functions or classes.

### Rule 2: Ensure That the Data Structure Is Immutable

Think of a data structure like a contract or guarantee. Within the context of a reconciler pattern, the actual and expected structures are set in memory at runtime.This guarantees that before a reconciliation, the structures are accurate. During the process of reconciling infrastructure, if the structure is changed, a new structure with the same guarantee must be created. A wise infrastructure application will enforce data structure immutability such that even if an engineer attempted to mutate a data structure, it wouldn’t work, or the program would error (or maybe even not compile).

The core component of an infrastructure application will be its ability to map a representation to a set of resources. A resource is a single task that will need to be run in order to fulfill the infrastructure requirements. Each of these tasks will be responsible for changing infrastructure in some way.

Basic examples could be deploying a new virtual machine, setting up a new network, or provisioning an existing virtual machine. Each of these units of work will be referred to as a resource. Each data structure should map to some number of resources. The application is responsible for reasoning about the structure and creating the set of resources. An example of how the API maps to individual resources can be seen in Figure 4-1.

Figure 4-1. Diagram to map a structure to resources

The reconciler pattern demonstrates a stable approach to working with a data structure as it mutates resources. Because the reconciler pattern requires comparing states of resources, it is imperative that data structures be immutable. This dictates that whenever the data structure needs to be updated, a new data structure must be created.

Be mindful of infrastructure mutations. Every time a mutation occurs, the actual data structure is then stale. A clever infrastructure application will be aware of this concern and handle it accordingly.

A simple solution would be to update the data structure in memory whenever a mutation occurs. If the actual state is updated with every mutation, then the reconciliation process can be observed as the actual state going through a set of changes over time until it ultimately matches the expected state and the reconciliation is complete.

### Rule 3: Keep the Resource Map Simple

Behind the scenes of the reconciler, the pattern is an implementation. An implementation is just a set of code that has methods to create, modify, and delete infrastructure. A program might have many implementations.

Each implementation will ultimately need to map a data structure to some set of resources. The set of resources will need to be grouped together in a logical way so that the program can reason about each of the resources.

Other than having the basic model of the resources created, you must give great attention to each resource’s dependencies. Many resources have dependencies on other resources, meaning that many pieces of infrastructure depend on other pieces to be in place. For example, a network will need to exist before a virtual machine can be placed in the network.

The reconciler pattern dictates that the simplest data structure for grouping resources should be used.

Solving the resource map problem is an engineering decision and might change for each implementation. It is important to pick a data structure carefully, as the reconciler needs to be stable and approachable from an engineering perspective.

Two common structures for mapping data are sets and graphs.

A set is a flat list of resources that can be iterated on. In many programming languages, these are called lists, sets, arrays, or the like.

A graph is a collection of vertices that are linked together via pointers. The vertex of a graph is usually a struct or a class, depending on the programming language. A vertex has a link to another vertex via a pointer defined somewhere in the vertex. A graph implementation can visit each of the vertices by hopping from one to the other via the pointer.

Example 4-5 is an example of a basic vertex in the Go programming language.

Example 4-5. Example vertex

```go
// Vertex is a data structure that represents a single point on a graph. // A single Vertex can have N number of children vertices or none at all. type Vertex struct {
Name string
    Children []*Vertex
}
```

An example of traversing the graph might be as simple as recursively iterating through each of the children. This traversal is sometimes called walking the graph.

Example 4-6 is an example of recursively visiting every vertex in the graph via a depth-first traversal written in Go.

Example 4-6. Depth-first traversal

```go
// recursiveWalk will recursively dig into all children, // and their children accordingly and echo the name of // the vertex currently being visited to STDOUT.
func recursiveWalk(v *Vertex){
fmt.Printf("Currently visiting vertex: %s\n", v.Name) for _, child := range v.Children {
        recursiveWalk(child)
    }
}
```

At first, a simple implementation of a graph seems like a reasonable choice for solving the resource map, as dependencies can be handled by building the graph in a logical way. While a graph would work, it also introduces risk and complexity. The biggest risk with implementing a graph to map resources would be having cycles in the graph. A cycle is when one vertex of a graph points to another vertex via a more than one path, meaning that traversing the graph is an endless operation.

A graph can be used when necessary, but for most cases, the reconciler pattern should be mapped with a set of resources, not a graph. Using a set allows the reconciler to iterate through the resources procedurally and offers a linear approach to solving the mapping problem. Furthermore, the process of undoing or deleting infrastructure is as simple as iterating through the set in reverse.

### Rule 4: Make the Actual State Match the Expected State

The guarantee offered in the reconciler pattern is that the user gets exactly what was intended or an error. This is a guarantee that an engineer who is consuming the reconciler can rely on. This is important, as the consumer shouldn’t have to concern themselves with validating that the reconciler mutation was idempotent and ended as expected. The implementation is ultimately responsible for addressing this concern.With the guarantee in place, using the reconciler pattern in more complex operations, such as a controller or operator, is now much simpler.

The implementation should, before returning to the calling code, check that the newly reconciled actual data structure matches the originally expected data structure. If it does not, it should error. The consumer should never concern themselves with validating the API and should be able to trust the reconciler to error if something goes wrong.

Because the data structures are immutable and the API will error if the reconciler pat‐tern is not successful, we can put a high level of trust in the API. With complex systems, it is important that you are able to trust that your software works or fails in predictable ways.

## The Reconciler Pattern’s Methods

With the information and rules of the reconciler patterns we just explained, let’s look at how some of those rules have been implemented. We will do this by looking at the methods needed for an application that implements the reconciler pattern.

The first method of the reconciler pattern is GetActual(). This method is sometimes called an audit and is used to query for the actual state of infrastructure. The method works by generating a map of resources, then procedurally calling each resource to see what, if anything, exists. The method will update the data structure based on the queries and return a populated data structure that represents what is actually running.

A much simpler method, GetExpected(), will read the intended state of the world from the data store. In the case of the infrastructure.yaml example (Example 4-4), GetExpected() would simply unmarshal this YAML and return it in the form of the data structure in memory. No resource auditing is done at this step.

The most exciting method is the Reconcile() method, in which the reconciler implementation will be handed the actual state of the world, as well as the expected state of the world.

This is the core of the intent-driven behavior of the reconciler pattern. The underlying reconciler implementation would use the same resource mapping logic used inGetActual() to define a set of resources. The reconciler implementation would then operate on these resources, reconciling each one independently.

It is important to understand the complexity of each of these resource reconciliation steps. The reconciler implementation must work in two ways.

First, get the resource properties from the desired and actual state. Next, apply changes to the minimal set of properties to make the actual state match the desired state.

If at any time the two representations of infrastructure conflict, the reconciler implementation must take action and mutate the infrastructure. After the reconciliation step has been completed, the reconciler implementation must create a new representation and then move on to the next resource. After all the resources have been reconciled, the reconciler implementation returns a new data structure to the caller of the interface. This new data structure now accurately represents the actual state of the world and should have a guarantee that it matches the original actual data structure.

The final method of the reconciler pattern is the Destroy() method. The wordDestroy() was intentionally chosen over Delete() because we want the engineer to be aware that the method should destroy infrastructure, and never disable it. The implementation of the Destroy() method is simple. It uses the same resource mapping as defined in the preceding implementation methods but merely operates on the resources in reverse.

### Example of the Pattern in Go

Example 4-7 is the reconciler pattern defined in four methods in the Go programming language.

Don’t worry if you don’t know Go. The pattern can easily be implemented in any language. We just use Go because it clearly defines the input and output type of each method. Please read the comments for each method, as it defines what each method needs to do, and when it should be used.

Example 4-7. The reconciler pattern interface

```go
// The reconciler interface below is an example of the reconciler pattern.
// It should be used whenever a user intends on mutating infrastructure based on a // state that might have changed over time.
type Reconciler interface {
    // GetActual takes no arguments for input and returns a populated data
    // structure as well as a possible error. The data structure should
    // contain a complete representation of the infrastructure.
    // This is sometimes called an audit. This method
    // should be used to get a real-time representation of what infrastructure is
    // in existence.
GetActual() (*Api, error)
    // GetExpected takes no arguments for input and returns a populated data
    // structure that represents what infrastructure an operator has declared to
    // exist, as well as a possible error. This is sometimes called expected or
    // intended state. This method should be used to get a real-time representation
    // of what infrastructure an operator intends to be in existence.
GetExpected() (*Api, error)
    // Reconcile takes two arguments.
    // actualApi is a populated data structure that is returned from the GetActual
    // method. expectedApi is a populated data structure that is returned from the
    // GetExpected method. Reconcile will return a populated data structure that is
    // a representation of the new "actual" state, as well as a possible error.
    // By definition, the data structure returned here should match
    // the data structure returned from the GetExpected method. This method is
    // responsible for making changes to infrastructure.
    Reconcile(actualApi, expectedApi *Api) (*Api, error)
    // Destroy takes one argument.
// actualApi is a populated data structure that is returned from the GetActual
// method. Destroy will return a populated data structure that is a
// representation of the new "actual" state, as well as a possible error. By
// definition, the data structure returned here should match
// the data structure returned from the GetExpected method.
Destroy(actualApi *Api) (*Api, error)
}
```

## The Auditing Relationship

As time progresses, the last audit of our infrastructure becomes stale, increasing the risk that our representation of infrastructure is inaccurate. So the trade-off is that an operator can exchange frequency of audits for the accuracy of infrastructure representation.

A reconciliation is implicitly an audit. If nothing has changed, the reconciler will detect that nothing needs to be done, and the operation becomes an audit, validating that our representation of our infrastructure, is accurate.

Furthermore, if there happens to be something that has changed in our infrastructure, the reconciler will detect the change and attempt to correct it. Upon completion of the reconcile, the state of the infrastructure is guaranteed to be accurate. So implicitly, we have audited the infrastructure again.

Auditing and Reconciler Pattern in Configuration Management

Infrastructure engineers may be familiar with the reconciler pattern from configuration management tools, which use similar methods to mutate operating systems. The configuration management tool is passed a set of resources to manage from a set of manifests or recipes defined by the engineer.

The tool will then take action on the system to make sure the actual state and desired state match. If no changes are made, then a simple audit is performed to make sure the states match.

The reason configuration management is not the same thing as cloud native infra‐structure applications are because of configuration management traditionally abstract single nodes and do not create or manage infrastructure resources.

Some configuration management tools are extending their use in this space to varying degrees of success, but they remain in the category of infrastructure as code and not the bidirectional relationship that infrastructure as the software provides.

A lightweight and stable reconciler implementation can yield powerful results that are reconciled quickly, giving the operator confidence in accurate infrastructure representation.

### Using the Reconciler Pattern in a Controller

Orchestration tooling such as Kubernetes offers a platform in which we can run applications conveniently. The idea of a controller is to serve a control loop for an intended state. Kubernetes is built on this fundamental. The reconciler pattern makes it easy to audit and reconcile objects controlled by Kubernetes.

Imagine a loop that would endlessly flow through the reconciler pattern in the following steps:

1. Call GetExpected() and read from a data store the intended state of infrastructure.
2. Call GetActual() and read from an environment to get the actual state of infrastructure.
3. Call Reconcile() and reconcile the states.

The program that implemented the reconciler pattern in this way would serve as a controller. The beauty of the pattern becomes immediately evident since it’s easy to see how small the program for the controller itself would have to be.

Furthermore, making a change to the infrastructure is as simple as mutating the state store. The controller will read the change the next time GetExpected() is called and trigger a reconcile. The operator in charge of the infrastructure can rest assured that astable and reliable loop is running quietly in the background, enforcing her will across her infrastructure environment. Now an operator manages infrastructure by managing an application.

> The goal seeking behavior of the control loop is very stable. This has been proven in Kubernetes where we have had bugs that have gone unnoticed because the control loop is fundamentally stable and will correct itself over time.
>
> If you are edge triggered you run a risk of compromising your state and never being able to re-create the state. If you are level triggered the pattern is very forgiving and allows room for components not behaving as they should be rectified. This is what makesKubernetes work so well.
>
> —Joe Beda, CTO of Heptio

Destroying infrastructure is now as simple as notifying the controller that we wish to destroy infrastructure. This could be done in a number of ways. One way would be to have the controller respect a disabled state file. This could be represented by flipping a bit from on to off.

Another way could be by deleting the content of the state. Regardless of how an operator chooses to signal a Destroy(), the controller is ready to call the convenientDestroy() method.

## Conclusion

Infrastructure engineers are now software engineers, tasked with building advanced and highly distributed systems—and working backward. They must write software that manages the infrastructure they are responsible for.

While there are many similarities between the two disciplines, there is a lifetime of learning the trade of engineering infrastructure management applications. Hard problems, such as bootstrapping infrastructure, continually evolve and require engineers to keep learning new things. There is also an ongoing need to maintain and optimize infrastructure that is sure to keep engineers employed for a very long time.

The chapter has equipped the user with powerful patterns and fundamentals in map‐ping ambiguous API structures into granular resources. The resources can be applied to your local data center, on top of a private cloud, or in a public cloud.

Understanding the basics of how these patterns work is critical to building reliable infrastructure management applications. The patterns set out in this chapter are intended to give engineers a starting point and inspiration for building declarative infrastructure management applications.

There is no right or wrong answer in building infrastructure management applications, so long as the applications adhere to the Unix philosophy: “Do one thing. Do it well.”