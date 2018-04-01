# Appendix A Patterns for Network Resiliency

Applications need to be resilient when running in a cloud environment. One important area especially prone to failure is network communications. One common pattern for adding network resiliency is to create a library that is imported into applications, which provides the network resiliency patterns described in this appendix. However, imported libraries become difficult to maintain for services written in many languages, and when new versions of the network library are released, it puts an additional burden on applications to test and redeploy.

Instead of making applications handle network resiliency logic, it is possible to put a proxy in place that can act as a layer of protection and enhancement for applications.A proxy has the advantage of sheltering the applications from needing additional complex code and minimizing developer effort for initial and ongoing development.

Network resiliency logic can be handled in the connection layer(physical or SDN), in the application, or via a transparent proxy.While proxies are not part of the traditional network stack, they can be used to transparently manage network resiliency for the applications.

Transparent proxies can run anywhere in the infrastructure, but are more beneficial the closer they are to the applications. They also need to be as comprehensive as possible in protocols and what Open Systems Interconnection model (OSI model) layers they can proxy.

Proxies play an active role in infrastructure resiliency by implementing the following patterns:

- Load balancing
- Load shedding
- Service discovery
- Retries and deadlines
- Circuit breaking

Proxies can also be used to add functionality to applications. Some features include:

- Security and authentication
- Routing (ingress and egress)
- Insight and monitoring

## Load Balancing

There are many ways to load balance an application and plenty of reasons why you should always put a load balancer in front of cloud native applications:

DigitalOcean explains some good reasons in “5 DigitalOcean Load Balancer UseCases”:

- Horizontal scaling
- High availability
- Application deployments
- Dynamic traffic routing

Coordinated transparent proxies (e.g., envoy and linkerd) are one way to load balance applications. Some benefits of having a transparent proxy handle load balancing are:

- A view of requests for all endpoints allows for better load-balancing decisions.
- Software-based load balancers allow flexibility in picking the right way to balance the load.

Transparent proxies don’t have to blindly pass traffic to the next router. As Bunyan points out in their blog post “Beyond Round Robin: Load Balancing for Latency,” they can be centrally coordinated to have a broader view of the infrastructure. This allows the load balancing to globally optimize traffic routing instead of only locally optimizing for quick packet handoffs.

With more knowledge about endpoints and which services are making and receiving requests, the proxy can be more rational about where to send traffic.

## Load Shedding

The Site Reliability Engineering book explains that load shedding is not the same as load balancing. While load balancing tries to find the correct backend to send traffic, load shedding intentionally drops traffic if the application cannot take the request.1

By dropping the load to protect the application instance, you can ensure the applications don't restart or are forced into unfavorable conditions. Dropping a request is much faster than waiting for timeouts and requiring applications to restart.

Load shedding can help protect application instances when something is broken or there is too much traffic. When things are working properly, the application should discover other dependent services with service discovery.

## Service Discovery

Service discovery is usually handled by the orchestration system running the services.Transparent proxies can tie into the same data and provide additional features.

Proxies can enhance standard service discovery by tying multiple sources together(e.g., DNS and a key-value database) and presenting them in a uniform interface.This allows implementors to change their backend without rewriting all application code. If service discovery is handled outside of the application, it may be possible to change the service discovery tooling without rewriting any application code.

Because the proxy can have a more holistic view of requests in the infrastructure, it can decide when endpoints are healthy or not. This works in conjunction with other features, such as load balancing and retries to get traffic routed to the best endpoint.

Proxies can also take additional metadata into account when allowing services to discover each other. They can implement logic, such as node latency or “distance,” to make sure the correct service is discovered for a request.

## Retries and Deadlines

Typically an application would use built-in logic to know what to do with a failed request to an external service. This can also be handled by a proxy without additional application code.

Proxies intercept all ingress and egress traffic to an application and route the requests. If an outgoing request comes back as failed, the proxy can automatically retry without needing to involve the application. If the request returns for any other reason, the proxy can process appropriately, based on rules in its configuration.

This is great, so long as the application is resilient to latency. If not, the proxy should return the failure notice based on an application’s deadline.

Deadlines allow applications to specify how long a request is allowed to take. Because the proxy can “follow” requests to their destination and back, it can enforce deadline policies across all applications that use the proxy.

When a deadline has been exceeded, the failure is returned to the application, and it can decide the appropriate action. An option may be to degrade the service, but the application may also choose to send an error back to the user.

## Circuit Breaking

The pattern is named for the same circuit breakers in-home wiring. Circuits default toa “closed” state when everything is working normally, and allow traffic to flow through the breaker. When failures are detected, the circuit “opens” and breaks the flow of traffic.

> The Retry Pattern enables an application to retry an operation in the expectation that it will succeed. The Circuit Breaker pattern prevents an application from performing an operation that is likely to fail.
>
> —Alex Homer, Cloud Design Patterns: Prescriptive Architecture Guidance for Cloud Applications

A broken circuit can be to an individual endpoint or an entire service. Once it’s open, no traffic will be sent, and all attempts to send traffic will immediately return as failed.

Unlike home circuits, even in an open state, the proxy will test the failed endpoint.Failed endpoints can be put in a “half-open” state when detected to be available again after a failure. This state will send small amounts of traffic until the endpoint is marked as failed or healthy.

This pattern can make applications much faster by failing fast and routing only to healthy endpoints. By continually checking endpoints, the network can self-heal and intelligently route traffic.

In addition to these resiliency features, proxies can enhance applications in the following ways.

## TLS and Auth

Proxies can terminate transport layer security (TLS) or any other security supported by the proxy. This allows for the security logic to be centrally managed and not re-implemented in every application. New security protocols or certificates can then be updated across the infrastructure without applications needing to be redeployed.

The same is true for authentication. However, authorization should still be managed by the application because it is usually a more fine-grained, application-specific feature. User session cookies can be validated by the proxy before the application oversees them. This means only authenticated traffic will be seen by the application.

This will not only save time for the application but can prevent downtimes through certain types of abuses as well.

## Routing (Ingress and Egress)

When proxies run in front of all your applications, they control traffic going in and out of the application. They can also manage the traffic going in and out of the cluster.

Just as reverse proxies can be used to route to backends in an N-tier architecture, the service proxies can be exposed to traffic outside the cluster and used to route requests coming in. This is another data point the proxies can use to know where traffic is coming from and where it is being sent.

Having a reverse proxy with insight into all the service-to-service communication means route choices can be better informed than traditional reverse proxies.

## Insight and Monitoring

With all of this knowledge about traffic flow inside the infrastructure, the proxy system can expose metrics about individual endpoints and cluster-wide views of traffic.These data points have traditionally been exposed in proprietary network systems or difficult-to-automate protocols (e.g., SNMP).

Because the proxies know immediately when endpoints are unreachable, they are the first to know when the endpoint is unhealthy. The orchestration system can also check application health, but the application may not know it is unhealthy to report the correct status to the orchestration tool. With knowledge about all endpoints serving the same service, proxies can also be the best place to monitor service health.