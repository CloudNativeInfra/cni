# Chapter 6 Testing Cloud Native Infrastructure

Infrastructure supports applications. Being able to trust software is critical to success in engineering. What if every time you typed the ls command in a terminal a random action happened? You would never trust ls and would find a different way to list files in a directory.

We need to be able to trust our infrastructure. This chapter is aimed at opening up the ideologies on trusting and verifying infrastructure. The practices we will describe are designed to increase confidence in infrastructure for applications and engineers.

The practice of testing software is quite common in the software engineering space today. The practice of testing infrastructure, however, is virtually undefined.

This means that of all lessons in this book, this should be the most exciting! There is room for engineers, like you, to make a fantastic impact in this space.

Testing software is the practice of proving that something works, that it doesn’t fail, and that those conditions remain true in a variety of edge cases. So if we apply the same paradigm to testing infrastructure, our goals for testing are as follows:

1. Prove the infrastructure works as intended.
2. Prove the infrastructure isn’t failing.
3. Prove that both conditions are true in a variety of edge cases.

Measuring whether or not infrastructure is working requires us to first define what working means. By now you should feel comfortable with the idea of representing infrastructure and engineering applications to manage the representation.

A user defining an infrastructure API should spend time focusing on creating a sane API that creates working infrastructure. For instance, it would be foolish to have an API that defines virtual machines, and no network information to run them in. You should create useful abstractions in the API and then use ideas laid out in Chapters 3and 4 to make sure the API creates the correct infrastructure components.

We have already begun to develop a mental model of what a sanity check of our infrastructure should look like just by defining the API. This means we can flip the logic and imagine the opposite case, which would be everything excluding what is in the original mental model.

The practice of defining basic sanity tests for infrastructure is a worthwhile endeavor.So the first step in testing infrastructure is to prove your infrastructure exists as intended and that nothing exists contrary to the original intent.

In this chapter we explore infrastructure testing and lay the groundwork for a new world order of testing tools.

## What Are We Testing?

Before we can begin writing a line of code, we must first identify our concerns that will need to be tested.

Test-driven development is a common practice in which testing comes first. The tests are written to demonstrate the concerns of testing and will implicitly fail on day one. The development cycle is aimed at making the tests pass; that is, the software is developed to address the requirements defined in each test. This is a powerful practice that helps software stay focused and helps engineers gain confidence in their software and thus their tests.

This is a philosophical question that can be answered in a number of ways. Having a good idea of what we need to prove true and not false is imperative to building trust‐worthy infrastructure. For instance, if there is a business concern that relies on infrastructure being in place, it should be tested. More importantly, many businesses rely on infrastructure not only being in place, but also fixing itself if something goes wrong.

Identifying the problem space that your infrastructure needs to fill represents the first round of testing that will need to be written.

Planning for the future is another important aspect of infrastructure testing but should be taken lightly. There is a thin line between being sufficiently forward-looking and over engineering. If in doubt, stick with the bare minimum amount of testing logic.

After we have a solid understanding of what we need to demonstrate with our tests, we can look at implementing a testing suite.

## Writing Testable Code

The rules of the reconciler pattern are not only aimed at creating a clean infrastructure application, but also explicitly designed to encourage testable infrastructure code.

This means that in every major step in our application we always fall back on creating a new object of the same type, which means every major component of our underlying system will speak the same input and output. Using the same inputs and outputs makes it easier to programmatically test small components of your software. Tests will ensure your components are working as expected.

However, there are many other valuable lessons to abide by while writing infrastructure-testing code. We will take a look at a concrete example of testing infrastructure in a hypothetical scenario. As we walk through the scenario, you will learn the lessons on testing infrastructure code.

We will also call out a handful of rules that engineers can abide by as they begin writing testable infrastructure code.

### Validation

Take a very basic infrastructure definition such as Example 6-1.

Example 6-1. infrastructure.json

```json
{
    "virtualMachines": [{
        "name": "my-vm",
        "size": "large",
        "localIp": "192.168.1.111",
        "subnet": "my-subnet"
    }],
    "subnets": [{
        "name": "my-subnet",
        "cidr": "10.0.100.0/24"
    }]
}
```

It should be obvious what the data is aiming to accomplish: ensure a single virtual machine called my-vm of an arbitrary size large with an IP address of 192.168.1.111. The data also alludes to ensuring a subnet with the name my-subnet that will house the virtual machine my-vm.

Hopefully you caught what is wrong with this data. The virtual machine has been given an IP address that falls outside of the available CIDR range for the subnet.

Running this particular data against an application should result in a failure, as the virtual machine will be effectively useless on the network. If our application was built to blindly allow any data to be deployed, we would create infrastructure that would network. While we should write a test to ensure that the new virtual machine is able to route on the network, there is something else we can do to help harden our application and make testing much easier.

Before our application entertains the idea of processing the input, we could first attempt to validate the input. This is a common practice in software engineering.

Imagine if, instead of blindly deploying this infrastructure, we first attempted to validate the input. At runtime our application could easily detect that the IP address for the virtual machine will not work in the subnet that the virtual machine is attached to. This would prevent the input from ever reaching our infrastructure environment.With the knowledge that the application will intentionally reject invalid infrastructure representation, we can write happy and sad tests to ensure this behavior is achieved.

A happy test is a test that exercises the positive case of a condition.In other words, it is a test that sends a valid API object to the application and ensures the application accepts the valid input. A sadist is a test that exercises the opposite case, or the negative case of a condition. Example 6-1 is an example of a sad test that sends an invalid API object to the application and ensures the application rejects the invalid input.

This new pattern makes testing infrastructure very quick, and usually inexpensive.An engineer can begin to develop a large arsenal of happy and sad tests for even the most bizarre application input. Furthermore, the testing collection can grow over time; in the unfortunate scenario where an impostor API object trickles through to the environment, an engineer can quickly add a test to prevent it from happening again.

Input validation is one of the most basic things to test. By writing simple validation into our application that checks for sane values, we can begin to filter out the input of our program. This also gives us an avenue in which it is easy to define meaningful errors and return an error quickly.

Validation offers confidence without making you wait for infrastructure to be mutated. This creates a faster feedback loop for engineers developing against the API.

### Entering Your Codebase

It is important that we continue to cater to ourselves and write code that is easily testable. An easy mistake that can be costly downstream is to engineer an application around proprietary input. Proprietary input is input that is relevant at only one point in the program, and the only way to get the desired input is to linearly execute the program. Writing code linearly in this fashion makes sense to the human brain but is one of the hardest patterns to effectively test, especially when it comes to testing infrastructure.

An example of getting into trouble with proprietary input is as follows:

1. A call to function DoSomething() returns Something{}.
2. Something{} is passed to the function NextStep(Something) and SomethingElse{} is returned.
3. SomethingElse{} is passed to the function FinalStep(something else) and true or false is returned.

The problem here is that in order to test the FinalStep() function we first need to traverse steps 1 and 2. In the case of testing, this introduces complexity and more points of failure; it very well might not even work within the context of a test execution.

A more elegant solution would be to structure the code in such a way that final step() could be called on the same data structure the rest of the program is using:

1. Code initializes GreatSomething{}, which implements the method great something.DoSomething().
2. GreatSomething{} implements the method great something.NextStep().
3. GreatSomething{} implements the method great something.FinalStep().

From a testing perspective, we can populate GreatSomething{} for any step we wish to test, and call the methods accordingly. The methods in this example are now responsible for acting on the memory defined in the object they are extending. This is different than the last approach, where ad hoc in-memory structures were passed into each function.

This is a much more elegant design, since a test engineer can synthesize the memory for any step along the way easily and will only ever have to concern themselves with learning one representation of the same data. This is much more modular, and we can home in on any failures if they are detected quickly.

As you begin to write the software that makes up your application, remember that you will need to jump into your codebase at many points during the traditional run‐time timeline. Structuring your code to make it easy to enter the codebase at any point, and therefore test the system internally, is critical. In situations like this, you can be your own best friend or worst enemy.

## Self-Awareness

> Tell me how you measure me, and I will tell you how I will behave.
>
> —Eliyahu M. Goldratt

Pay attention to your confidence levels while writing your code and writing your tests. Self-awareness is one of the most important parts of software engineering and is frequently one of the most overlooked.

The ultimate goal of testing is to increase confidence in your application. In the realm of infrastructure, our goal is to increase our confidence that our infrastructure does what we want it to do.

With that being said, there is no right or wrong approach to testing infrastructure. Its easy to get caught up in metrics like code coverage or unit testing every function in your application. But these introduce false confidence.

Code coverage is the act of programmatically measuring how much of your codebase is being exercised by tests. This metric can be used as a primitive datapoint, but it is critical to understand that even a codebase with 100 percent coverage could still be subject to an extreme outage.

If you measure your tests by code coverage, then engineers will write code that can more easily be covered by tests instead of code that is more appropriate for the task.Dan Ariely sums up human behavior this way in his article “You Are What You Measure” for the Harvard Business Review:

Human beings adjust behavior based on the metrics they’re held against. Anything you measure will impel a person to optimize his score on that metric. What you measure is what you’ll get. 

The only metric we should ever be measuring is our confidence that our infrastructure is working as intended and that we can demonstrate that.

Measuring confidence can be close to impossible. But there are ways that engineers have pulled meaningful data sets out of psychological and emotional spaces.

By asking ourselves questions like the following, we can record our answers overtime:

- Am I worried this won’t work? 
- Can I be certain this will do what I think it will do?
- What happens if somebody changes this file?

One of the most powerful techniques in pulling data out of opinionated questions is to compare levels from previous experiences. For example, an engineer could make a statement like the following, and the rest of the team very quickly understands what they are trying to relay:

I am significantly more worried about this code-breaking than I was worried about the release last quarter.

Now, based on the team’s prior experience, we can begin to develop a scale for our confidence levels, with the 0 end of the scale being an experience when the team had virtually no confidence, and the upper end of the scale being a time they felt extremely confident. As we understand what worries us about our application, understanding what needs to be tested to increase confidence is simple.

## Types of Tests

Understanding the types of tests, and how they are intended to be used, will help an engineer increase the confidence of their infrastructure applications. These tests are not necessary to write, and there is no right or wrong approach to using them. The only concern is that we trust our application to do what we want it to do.

### Infrastructure Assertions

In software engineering, a powerful concept is the assertion, which is a way of force‐fully determining if a condition is true. There are many successful frameworks that have been developed that use assertions to test software. An assertion is a tiny function that will test if a condition is true. These functions can be used in various testing scenarios to demonstrate that concepts are working, and hopefully introduce confidence.

Throughout the rest of the chapter we will be referring to infra‐structure assertions. You will need to have a basic understanding of what these assertions look like, and what they hope to accomplish.You will also need a basic understanding of the Go programming language to fully realize what these assertions are doing.

There is a need in the infrastructure space for asserting that our infrastructure works.Building out a library of these assertion functions would be a worthwhile exercise for your project. The open source community could also benefit from having this toolkit to test infrastructure.

Example 6-2 shows an assertion pattern in the Go programming language. Let’s imagine we would like to test if a virtual machine can resolve public hostnames and then route to them.

Example 6-2. assertNetwork.go

```go
type VirtualMachine struct { localIp string
}
func (v *VirtualMachine) AssertResolvesHostname( hostname string,
expectedIp string,
message string) error { // Logic to query DNS for a hostname,
    // and compare to the expectedIp
return nil
}
func (v *VirtualMachine) AssertRouteable( hostname string,r
port int,
message string) error {
// Logic to verify the virtualMachine can route
    // to a hostname on a specific port
return nil
}
func (v *VirtualMachine) Connect() error {
// Logic to connect to the virtual machine to run the assertions return nil
}
func (v *VirtualMachine) Close() error {
// Logic to close the connection to the virtual machine return nil
}
```

In this example we stub out two assertions as methods on the VirtualMachine{}struct. The method signatures are what we will be focusing on in this demonstration.

The first method, AssertResolvesHostname(), demonstrates a method that would be used to check if a given hostname resolves to an expected IP address. The second method, AssertRouteable(), demonstrates a method that would be used to check if a given hostname was routable on a specific port.

Notice how the VirtualMachine{} struct has the member local IP defined. Also notice how the VirtualMachine{} struct has a Connect() function, as well as a Close() function. This is so that the assertion framework could run this assertionsfrom within the context of the virtual machine. The test could run from a system outside the infrastructure environment and then connect to the virtual machine within the environment to run infrastructure assertions.

In Example 6-3, we demonstrate how this would look and feel to an engineer writing tests on their local system using a Go test.

Example 6-3. network_test.go

```go
func TestVm(t *testing.T) {vm := VirtualMachine{

localIp: "10.0.0.17",

if err := vm.Connect(); err != nil {
    t.Fatalf("Unable to connect to VM: %v", err)
}

defer vm.Close()

if err := vm.AssertResolvesHostname("google.com", "*",

"google.com should resolve to any IP"); err != nil {t.Fatalf("Unable to resolve hostname: %v", err)

}

if err := vm.AssertRouteable("google.com", 443,

"google.com should be routable on port 443"); err != nil {t.Fatalf("Unable to route to hostname: %v", err)

}}
```

The example uses the built-in testing standards in the Go programming language, meaning that this function will be executed as part of a normal go test run against the Go tests in your application. The testing framework will test all files with a name that ends in _test.go and test all functions with a signature name that starts withTestXxx. The framework will also pass in the *testing.T pointer to each of the functions defined in that way.

This simple test will use the assertion library we defined earlier to complete a few steps:

1. Attempt to connect to a virtual machine that should be reachable on 10.0.0.17.
2. From the virtual machine, attempt to assert that the virtual machine can resolve
   google.com and that it returns some IP address.
3. From the virtual machine, attempt to assert that the virtual machine can route to google.com on port 443.
4. Close the connection to the virtual machine.

This is an extremely powerful program. It introduces a high level of confidence that our infrastructure is working as intended. It also introduces an elegant palette for engineers to define tests without having to worry about how they will be run.

The open source community is in desperate need of infrastructure testing frameworks like this. Having a standardized and reliable way of defining infrastructure tests would be a welcome addition to an engineer’s toolbox.

### Integration Testing

Integration tests are also known as end-to-end (e2e) tests. These are long-running tests that exercise the system in the way it is intended to be used in production. These are the most valuable tests in demonstrating reliability and thus increasing confidence.

Writing an integration testing suite can be fun and rewarding. In the case of integration testing an infrastructure management application, the tests would perform a sweep of the life cycle of infrastructure.

A simple example of a linear integration testing suite would be as follows:

1. Define a commonly used infrastructure API.
2. Save the data to the application’s data store.
3. Run the application, and create infrastructure.
4. Run a series of assertions against the infrastructure.
5. Remove the API data from the application’s data store.
6. Ensure the infrastructure was successfully destroyed.

At any step along the way, the test might fail, and the testing suite should clean up whatever infrastructure it has mutated. This is one of the many reasons it is so important to test that destroying infrastructure works as expected.

The test gives us confidence that the application will create and destroy infrastructures intended and that it works as expected. We can increase the number of assertions that are run in step 4 over time and continue to harden our suite.

The integration testing harness is potentially the most powerful context in which we can test our infrastructure. Without the integration testing harness, running smaller tests like unit tests wouldn’t offer very much value.

### Unit Testing

Unit testing is a fundamental part of testing a system and exercising its components individually. The responsibility of a unit test is intended to be small and discreet. Unit testing is a common practice in software engineering, and thus will be a part of infrastructure engineering.

In the case of writing infrastructure tests, testing one component of the system is difficult. Most components of infrastructure are built on top of each other. Testing that software mutates infrastructure accordingly usually requires mutating infrastructure to test and see if it worked. This process will usually involve a large portion of a system.

But that does not mean it’s impossible to write unit tests for an infrastructure management system. In fact, most of the assertions defined in the previous example are technically unit tests! Unit tests test only one small component, but they can be extremely helpful when used within the context of a larger integration testing system.

Unit tests are encouraged while you’re testing infrastructure, but remember that the context in which they run usually requires a fair amount of overhead. This overhead usually comes in the form of integration testing. Combining the small and discreet checks of unit testing with the larger, overarching patterns of integration testing gives infrastructure engineers a high level of confidence that their infrastructure is working as intended.

### Mock Testing

In software engineering, a common practice to synthesize systems is mock testing. In mock testing, an engineer writes or uses software that is designed to spoof, or fake, system.

A simple example would be taking an SDK that is designed to talk to an API and running it in “mock” mode. The SDK doesn’t send any data to the API but rather synthesizes what the SDK thinks the API should do in various situations.

The responsibility of ensuring that the mock software accurately reflects the system that it is synthesizing lies in the hands of the engineers developing the mock software.In some cases, the mock software is developed by the same engineers that develop the system it is mocking.

While there might be some mock tools that are kept up to date and are more stable than others, there is one universal truth in using mock systems to synthesize the infrastructure you plan to test: fake systems give you fake confidence.

Now, this rule may seem harsh. But it is intended to encourage engineers to not take the easy way out and to go through the practice of building a real integration suite to run their tests. While mock systems can be powerful, relying on them for the core of your infrastructure testing (and thus your confidence) is highly risky.

Most public cloud providers enforce quota limits for their resources. Imagine a test that was interacting with a system that had these hard limits on resources. The mock system might do its best to put a cap on resources—but without auditing the real system at runtime, the mock system would have no way of determining if your infrastructure would actually be deployed. In this case, your mock test would succeed.However, when the code is run in the real environment, it would break.

This is just one example of many that demonstrates why mutating real infrastructure and sending real network packets is far more reliable than using a mock system.Remember, the goal of testing is to increase confidence that your infrastructure will work as intended in a real environment.

This is not to say that all mock testing is bad. It is very important to understand the difference between mocking the infrastructure that you are testing and mocking another piece of the system for convenience.

The engineer will need to decide when is and isn’t appropriate to use a mock system.We just caution engineers from developing too much confidence in these systems.

### Chaos Testing

Chaos testing is probably the most exciting avenue of testing infrastructure that we will cover in this book. It’s testing to demonstrate that unpredictable events in infrastructure can occur, without jeopardizing the stability of the infrastructure. We do this demonstration by intentionally breaking and disrupting infrastructure, and measuring how the system responds to the catastrophe. As with all our tests, we will approach this as an infrastructure engineer.

We will be writing software that is designed to break production systems in unexpected ways. Part of building confidence in systems is understanding how and why they break.

> **Building Confidence at Google**
>
> One example of learning how systems break can be seen in Google’s DiRT (DisasterRecovery Training) program. The program is intended to help Google’s site reliability engineers stay familiar with the systems they support. In Site Reliability Engineering, they explain that the purpose of the DiRT program is because “being out of touch with production for long periods of time can lead to confidence issues, both in terms of over confidence and under confidence, while knowledge gaps are discovered only when an incident occurs.”

Unfortunately, it will not do the engineering team much good to just break production without having systems in place to measure the impact and recover from the catastrophe. Again, we will call on the infrastructure assertions defined earlier in the chapter. The tiny, single responsibility functions offer fantastic data points for measuring systems’ stability over time.

**Measuring chaos**

Let’s look at the AssertRouteable() function from Example 6-3 again. Imagine that we have a service that will connect to a virtual machine and attempt to keep the connection open for eternity. Every second the service will call the AssertRouteable() function and log the result. The data from this service is an accurate representation of the virtual machine’s ability to route on their network. As long as the virtual machine can route, the data would yield a straight, unchanged line on a graph as in Figure 6-1.

![f-6-1](../images/f-6-1.jpg)

Figure 6-1. Graph of AssertRoutable tests over time

If at any time the connection broke, or the virtual machine was no longer able to route, the graph data would shift, and we would see a change in the line on the graph.Over time, as the infrastructure repaired itself, the line on the graph would again stabilize, as seen in Figure 6-2.

![f-6-2](../images/f-6-2.jpg)

Figure 6-2. Failed and repaired AssertRoutable tests over time

The important dimension to consider here is time. Measuring chaos implicitly comes with the measurement of chaos over time.

We can expand our measurements very quickly. Imagine that the service that calledAssertRouteable() was now calling a set of 100 infrastructure assertions on the virtual machine. Also, imagine that we had 100 virtual machines that we were measuring.

This would yield roughly 1.0×104 assertions per second against our infrastructure.This enormous amount of data coming out of our infrastructure assertions allows us to create powerful graphs and representations of our infrastructure. Recording the data in a queryable format also allows for advanced after-chaos investigation.

With measuring chaos, it is important to have reliable measuring tools and services. It is also important to store the data from the services in meaningful ways so it can be referenced later. Storing the data in a log aggregator or another easily indexed datastore is highly encouraged.

The chaos of a system is inversely related to the reliability of the system. Therefore, itis a direct representation of the stability of the infrastructure we are evaluating. That means it is valuable to have the information graphed over time for an analysis when things break or when introducing change to know if it decreased the stability.

**Introducing chaos**

Introducing chaos into a system is another way of saying “intentionally breaking system.” We want to synthesize unexpected permutations of infrastructure issues that we might see in the wild. If we don’t intentionally inject chaos, the cloud provider, internet, or some system will do it for us.

> **Netflix’s Simian Army**
>
> Netflix introduced what it calls the Simian Army to cause chaos in its systems. Monkeys, apes, and other animals in the simian family are each responsible for causing chaos in different ways. Netflix explains how one of these tools, Chaos Monkey, works:
>
> This was our philosophy when we built Chaos Monkey, a tool that randomly disables our production instances to make sure we can survive this common type of failure without any customer impact. The name comes from the idea of unleashing a wild monkey with a weapon in your data center (or cloud region) to randomly shoot down instances and chew through cables—all the while we continue serving our customers without interruption. By running Chaos Monkey in the middle of a business day, in a carefully monitored environment with engineers standing by to address any problems, we can still learn the lessons about the weaknesses of our system, and build automatic recovery mechanisms to deal with them. So next time an instance fails at 3 a.m. on a Sunday, we won’t even notice.
>
> In terms of cloud-native infrastructure, the monkeys are great examples of infrastructure as software and utilizing the reconciler pattern. A major difference is that they are designed to destroy infrastructure in unexpected ways instead of creating and managing infrastructure predictably.

At this point you should have an infrastructure management application ready to use, or at least one in mind. The same infrastructure management application used to deploy, manage, and stabilize infrastructure can also be used to introduce chaos.

Imagine two very similar deployments.

The first, Example 6-4, represents working (or happy) infrastructure.

Example 6-4. infrastructure_happy.json

```json
{
    "virtualMachines": [{
        "name": "my-vm",
        "size": "large",
        "localIp": "10.0.0.17",
        "subnet": "my-subnet"
    }],
    "subnets": [{
        "name": "my-subnet",
        "cidr": "10.0.100.0/24"
    }]
}
```

We can deploy this infrastructure using the means set up in the environment. This infrastructure should be deployed, running, and stable. Just as before, it’s important to record your infrastructure tests over time; Figure 6-3 is an example of this. Ideally, the amount of tests you run should increase over time.

We decide we want to introduce chaos. So we create a copy of the original infrastructure management application, although this time we take a far more sinister approach to deploying infrastructure. We take advantage of our deployment tool’s ability to audit infrastructure and make changes to infrastructure that already exists.

![f-6-3](../images/f-6-3.jpg)

Figure 6-3. Successful tests measured over time

The second deployment would represent intentionally faulty infrastructure and still use the same identifiers (names) as the original infrastructure. The infrastructure management tool will detect existing infrastructure and change it. In the second example (Example 6-5), we change the virtual machine size to small and intention‐ally assign the virtual machine a static IP address 192.168.1.111 that is outside their range 10.0.100.0/24.

We know the workload on the virtual machine will not run on a small virtual machine, and we know the virtual machine will not be able to route on the network.This is the chaos we will be introducing.

Example 6-5. infrastructure_sad.json

```json
{
    "virtualMachines": [{
        "name": "my-vm",
        "size": "small",
        "localIp": "192.168.1.111",
        "subnet": "my-subnet"
    }],
    "subnets": [{
        "name": "my-subnet",
        "cidr": "10.0.100.0/24"
    }]
}
```

As the second infrastructure management application silently makes changes to the infrastructure, we can expect to see things break. The data on our graphs will begin fluctuating, as seen in Figure 6-4.

![f-6-4](../images/f-6-4.jpg)

Figure 6-4. Graph with network failures

Any applications on the virtual machine that have been altered should slowly fail if they don’t break entirely. The virtual machine’s memory and CPU are now overloaded. The shell is unable to fork new processes. The load average is well above 20. The system is approaching deadlock, and we can’t even access the virtual machine to see what is wrong because nothing can route to the impostor IP address.

As expected, the initial system will detect that something has changed in the underlying infrastructure and will reconcile it accordingly. It is important that the impostor system be taken offline, or else there is a risk of never-ending reconciliation between the two systems, which will compete to rectify the infrastructure in the way they were instructed to do so.

The beauty of this approach to introducing chaos is that we didn’t have to develop any extra tooling or spend any engineering hours writing chaos frameworks. We abused the original infrastructure management tool in a clever way to introduce a catastrophe.

Of course that may not always be a perfect solution. Unlike your production infrastructure applications, your chaos applications should have constraints to make sure they are beneficial. Some common constraints are the ability to exclude certain systems based on tags or metadata, not run chaos tests during off hours, and limit chaos to a certain percentage or type of system.

The burden of introducing random chaos now lies in an infrastructure engineers ability to randomize the workflow we just explored over time. Of course, the infrastructure engineer will also need to ensure that the data gathered from their experiment is offered in a digestible format.

## Monitoring Infrastructure

Along with testing infrastructure, we cannot forget to monitor what is actively running. Tests and familiar failure patterns can go a long way to give you confidence in your infrastructure, but it is impossible to test every way a system can fail.

Having monitoring that can detect anomalies not identified during testing and perform the correct actions is crucially important. With active monitoring of the live infrastructure, we can also increase our confidence that when something happens that is not recognized as “normal,” we will be alerted. Knowing when and how anomalies should alert a human is a topic of much debate.

There are lots of good resources that are emerging from the practice of monitoring infrastructure in a cloud native environment. We will not address the topics here, but you should start by reading Monitoring Distributed Systems: Case Studies from Google’sSRE Teams (O’Reilly) by Rob Ewaschuk and watching videos from the Monitoramaconference. Both are available for free online.

Whatever monitoring solution you implement, remember the cloud native approach to creating your monitoring rules. Rules should be declarative and stored as code.Monitoring rules should live with your application code and be available in a self-service manner. Don’t overcompensate for monitoring when testing and telemetry will likely fulfill most of your needs.

## Conclusion

Testing needs to introduce confidence and trust in the infrastructure so we gain confidence and trust in the applications we are supporting. If a testing suite does not offer confidence, its value should be in question. Try to remember that the tools and patterns suggested in this chapter are starting points and are designed to excite and engage an engineer working in this space. Regardless of the types of tests, or the framework that is used to run them, the most important takeaway is that an engineer can begin to trust their systems. As engineers, we typically gain trust by observing hands-on demonstrations that prove something is working as expected.

Furthermore, experimenting in production is not only OK, it is encouraged. You will want to make sure the environment was built for such experimentation and proper tracking is implemented so the testing does not go to waste!

Measuring reality is an important part of infrastructure development and testing. Being able to encapsulate reality from both an engineering perspective and an operating perspective is an important part of exercising infrastructure and thus gaining confidence that it works as intended.