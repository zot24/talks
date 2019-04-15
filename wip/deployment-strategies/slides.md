footer: powered by @zot24 🐉

# Deployment Strageties
## with Kubernetes

![](img/intro-roads.jpg)

^ Before we start questions: docker, containers, kubernetes, service mesh, istio

^ Choices, life it's full of them!

[.footer-style: alignment(right)]
<!--
---

![inline 130%](img/too-many-choices.jpg)

[.hide-footer]
-->
---

# What will you learn today? 📚

### You'll be able to understand the **different choices** we have when it comes to **operate** our applications *deployments*.

^ Notice the highlight words, I want to empahtise here that different choices => flexibility => operational responsability

^ => implies

[.hide-footer]

---

# Summary 🗓

- 🔖 Key concepts 
    > 🤾 **Resiliency & Reliability**
    > 📡 Liveness & Readiness
    > 🚀 Deploy != Release
- 👾 Deployment strategies
- 🔑 Take Away 

^ We're gonna start with few key concepts that are basics for each strategy and then dig deep into each of them with a live demo!

[.hide-footer]

---

# Resiliency 🤾‍‍

[.hide-footer]

---

# The ability of an app to recover from certain types of failure and yet remain functional from the customer perspective.

![inline 50%](img/springs.jpg)

^ Flexibility of our app when it comes to handle from errors, failures and sloweness

[.hide-footer]

---

# What does it means that an application is **reliabe**?

[.hide-footer]

---

# It *operates perfectly* all the time 👌

^ It's what we aim for, 100% operability

[.hide-footer]

---

# 🤔
## but ... how?

^ but how we actually achieve this?

[.hide-footer]

---

# 🤾‍‍

^ Any body?

[.hide-footer]

---

# 🤾‍‍ Resiliency

^ Throught resiliency
^ **Resiliency** can also be called **Recoverability**

[.hide-footer]

---

# Summary 🗓

- 🔖 Key concepts 
    - 🤾 Resiliency & Reliability
    - 📡 **Liveness & Readiness**
    - 🚀 Deploy != Release
- 👾 Deployment strategies
- 🔑 Take Away

^ Another important concept is: Liveness & Readiness

[.hide-footer]

---

# What is *liveness & readiness* about?
# 🙋‍

^ ... and therefore what is it?

^ example: If the process in your Container is able to crash on its own whenever it encounters an issue or becomes unhealthy, you do not necessarily need a liveness probe; the kubelet will automatically perform the correct action in accordance with the Pod’s restartPolicy.
^ 
^ If you’d like your Container to be killed and restarted if a probe fails, then specify a liveness probe, and specify a restartPolicy of Always or OnFailure.

[.hide-footer]

---

# Exposure and probe 🌞

^ Use this to improve slide eplanation https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes

^ This concepts will us to adopt different deployment strategies

[.hide-footer]

---

# Be able to know what the state of an *application* is, so that the ecosystem where **it lives** and which **manages** it can be able to do it's job better.


^ Let's see an example

[.hide-footer]

---

# Example[^1] 📡

[.code-highlight: none]
[.code-highlight: 2-3]
[.code-highlight: 6-7]
[.code-highlight: 10-11]
[.code-highlight: all]

```go
// Our app is not happy if we've got more than 100 goroutines running.
health.AddLivenessCheck("goroutine-threshold",
    healthcheck.GoroutineCountCheck(100))

// Our app is not ready if we can't resolve our upstream dependency in DNS.
health.AddReadinessCheck("upstream-dep-dns",
    healthcheck.DNSResolveCheck("upstream.example.com", 50*time.Millisecond))

// Our app is not ready if we can't connect to our database (`var db *sql.DB`) in <1s.
health.AddReadinessCheck("database",
    healthcheck.DatabasePingCheck(db, 1*time.Second))
```

[^1]: https://github.com/heptiolabs/healthcheck

^ Readiness

^ Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, you don’t want to kill the application, but you don’t want to send it requests either. 

^ Liveness

^ Will tell Kubernetes when to restart a Container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a Container in such a state can help to make the application more available despite bugs.

^ How the application communicate with it's environment and expose what's state it's in.

[.hide-footer]

---

# Summary 🗓

- 🔖 Key concepts 
    - 🤾 Resiliency & Reliability
    - 📡 Liveness & Readiness
    - 🚀 **Deploy != Release**
- 👾 Deployment strategies
- 🔑 Take Away

[.hide-footer]

---

# Deploy != Release 🚀

We deploy when we install our application into a particular environemnt, however we only release our application when we make it avaiabe to users

**Deployment** is what happens when you install some version of your software into a particular environment

**Release** is when you make a system or some part of it (for example, a feature) available to users

<!-- Stick with the concepts and differences between deploy and release
Explain the differentiation -->

[^1]: Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation

^ When talking about a deployment the production environment is often implied

^ Related with blue-green deployments and canary releasing

^ You can—and should—deploy your software to its production environment before you make it available to users, so that you can perform smoke testing and any other tasks such as waiting for caches to warm up. The job of smoke tests is to make sure that the deployment was successful, and in particular to test that the configuration settings (such as database connection strings) for the production environment are correct.

[.hide-footer]

---

# Summary 🗓

- 🔖 Key concepts 
    - 🤾 Resiliency & Reliability
    - 📡 Liveness & Readiness
    - 🚀 Deploy != Release
- 👾 **Deployment strategies**
- 🔑 Take Away

[.hide-footer]

---

# Deployment strategies 👾

- Recreate
- Ramped
- Blue/Green
- Canary
- A/B testing
- Shadow

[.hide-footer]

---

# Summary 🗓

- 🔖 Key concepts 
    - 🤾 Resiliency & Reliability
    - 📡 Liveness & Readiness
    - 🚀 Deploy != Release
- 👾 Deployment strategies
- 🔑 **Take Away**

[.hide-footer]

---

![inline 100%](img/cindy-tweet.png)

^ There is a lot of saying in this tweet comments however I meant to highlight what Cindy say, there is no much differcen between these two sentences: "works in staging" = "works in my machine" 

^ Testing in prodution without affecting customer experience it's possible and some of the previous explained strategies will allow us to achieve so

[.hide-footer]

---

# [fit] "works in *staging*" = "works in my *machine*" 

[.hide-footer]

---

# Take aways 🔑

- **Traffic Shadowing and Dark Launching** by *Daniel Byrant*
    - [http://bit.ly/2UOATS0](http://bit.ly/2UOATS0)

- **Four Priciples of Low-Risk Software Releases** by *Jez Humble*
    - [http://bit.ly/2W9wdX6](http://bit.ly/2W9wdX6)

- **Testing in Production, the safe way** by *Cindy Sridharan*
    - [http://bit.ly/2HJJYrJ](http://bit.ly/2HJJYrJ)

^ Decouple Deployment and Release

^ This is a slide with a lot of info it's not meant to be read at presentation time but later one to have some resources to research about the matter

^ If you want to get a deeper understanding of all these concepts and techniques this is the long "hard" way article that you "could" read

^ Short, easy to read and understand "take away" that you "should" read

[.hide-footer]

---

# Questions?

🙋‍ We can follow up on Thursday *Kubernetes Working Session**

[.hide-footer]

---

# Thank you 👏
## 🎙🦜 [https://github.com/zot24/talks](https://github.com/zot24/talks)

^ All the code and presentation for this talk it's on my G/H talks repo

[.footer-style: alignment(right)]

<!--
# Resources

- https://github.com/heptiolabs/healthcheck
- https://medium.com/platformer-blog/enable-rolling-updates-in-kubernetes-with-zero-downtime-31d7ec388c81
- https://linkerd.io/2/getting-started/
- https://github.com/ContainerSolutions/k8s-deployment-strategies
- https://container-solutions.com/kubernetes-deployment-strategies/
- https://thenewstack.io/deployment-strategies/
- https://medium.com/@cgrant/deployment-strategies-release-best-practices-6e557c3f39b4
- https://rollout.io/blog/deployment-strategy/
- https://cabforward.com/the-difference-between-reliable-and-resilient-software/
- https://docs.deckset.com/English.lproj/Presenting/01-presenter-notes.html
- https://speakerdeck.com/zot24/from-eval-to-prod-how-a-service-mesh-helped-us-build-production-cloud-native-services
- https://www.quora.com/What-is-a-dark-launch-in-terms-of-continuous-delivery-of-software
- https://twitter.com/copyconstruct/status/974530841190608897
- https://blog.getambassador.io/embrace-the-dark-side-of-api-gateways-traffic-shadowing-and-dark-launching-976984f9d094
- http://www.informit.com/articles/article.aspx?p=1833567&seqNum=2
- https://medium.com/@copyconstruct/testing-in-production-the-safe-way-18ca102d0ef1
- https://en.wikipedia.org/wiki/Smoke_testing_(software)
- https://en.wikipedia.org/wiki/Soak_testing
- https://blog.getambassador.io/embrace-the-dark-side-of-api-gateways-traffic-shadowing-and-dark-launching-976984f9d094
-->

<!--
# Shadow
# Dark Launches, shadow traffic/ mirror traffic?

[ADD PICTURE]

Dark Launches - https://www.quora.com/What-is-a-dark-launch-in-terms-of-continuous-delivery-of-software

> NOTE: You can achieve dark launches by adding a feature flag in your code or by deploying (not relasing) a newer version of your code with the feature you want and only reroute traffic to certain users

Dark launching is the practice of deploying the very first version of a service into its production environment, well before release, so that you can soak test it and find any bugs before you make its functionality available to users.

However, as our systems evolve, it would be nice to have a way to decouple the deployment of a new version of our software from the release of the features within it. In this way, we can deploy new versions of our software continuously, completely independently of the decision as to which features are available to which users. Feature toggles can perform this function.

source: http://www.informit.com/articles/article.aspx?p=1833567&seqNum=2

Soak testing involves testing a system with a typical production load, over a continuous availability period, to validate system behavior under production use.[1]
https://en.wikipedia.org/wiki/Soak_testing

In computer programming and software testing, smoke testing (also confidence testing, sanity testing,[1] build verification test (BVT)[2][3][4] and build acceptance test) is preliminary testing to reveal simple failures severe enough to, for example, reject a prospective software release. Smoke tests are a subset of test cases that cover the most important functionality of a component or system, used to aid assessment of whether main functions of the software appear to work correctly
https://en.wikipedia.org/wiki/Smoke_testing_(software) -->

^ There are two key concepts that will help us to improve our deployment pipeline: resiliency, and the distiction between a deployment and a release
