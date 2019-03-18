# Deployment Strageties (with Kubernetes)

^ Choices, life it's full of them!

^ There are two key concepts that will help us to improve our deployment pipeline: resiliency, and the distiction between a deployment and a release

---

# Resilient

The ability of an app to recover from certain types of failure and yet remain functional from the customer perspective.

^ Reliability: Is the target at which software designers have always aimed: perfect operation all the time. Reliability is the planned outcome. Resilience is how you achieve the outcome. Resiliency can also be called Recoverability.

---

# Deploy != Release

### Dark Launches, shadow traffic?

> NOTE: You can achieve dark launches by adding a feature flag in your code or by deploying (not relasing) a newer version of your code with the feature you want and only reroute traffic to certain users

To gradually release and test new features to a small set of their users before releasing to everyone.

`Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation`

Principle 2: Decouple Deployment and Release

Blue-green deployments and canary releasing are examples of applying the second of my four principles: decoupling deployment and release. Deployment is what happens when you install some version of your software into a particular environment (the production environment is often implied). Release is when you make a system or some part of it (for example, a feature) available to users.

You can—and should—deploy your software to its production environment before you make it available to users, so that you can perform smoke testing and any other tasks such as waiting for caches to warm up. The job of smoke tests is to make sure that the deployment was successful, and in particular to test that the configuration settings (such as database connection strings) for the production environment are correct.

Dark launching is the practice of deploying the very first version of a service into its production environment, well before release, so that you can soak test it and find any bugs before you make its functionality available to users.

However, as our systems evolve, it would be nice to have a way to decouple the deployment of a new version of our software from the release of the features within it. In this way, we can deploy new versions of our software continuously, completely independently of the decision as to which features are available to which users. Feature toggles can perform this function.

source: http://www.informit.com/articles/article.aspx?p=1833567&seqNum=2

Will help with https://en.wikipedia.org/wiki/Soak_testing and https://en.wikipedia.org/wiki/Smoke_testing_(software)

---

# Liveness & Readiness

This will help us to adopt different deployment strategies

^ Show how to Go application implement this and what library are they using for to do so
^ Mention as well that this is a good practise to adopt from the developer in order to expose the healthiness of their applications

https://github.com/heptiolabs/healthcheck
https://medium.com/platformer-blog/enable-rolling-updates-in-kubernetes-with-zero-downtime-31d7ec388c81

---

# Deployment procedure

- Recreate
- Ramped
- Blue/Green
- Canary
- A/B testing
- Shadow
- Dark Launches - https://www.quora.com/What-is-a-dark-launch-in-terms-of-continuous-delivery-of-software

^ I'll do it "the non standar way" and run a demo first for each case and then after we can review the basic concept behind them

---

# Recreate
## DEMO

---

# Recreate
## Recapitulate

Development focus stragety

[ADD PICTURE]

---

# Ramped

[ADD PICTURE]

---

# Ramped

[ADD PICTURE]

^ Circuit Breaking: Because we are hitting it so frequently there could be some request that are caught in between the LB and the pod that it's being destryoed and those will fail, in this case a small % of all request that's where a service mesh could help to retry that failed request using the circuit breaking principle

---

# Blue / Green

[ADD PICTURE]

---

# Canary

[ADD PICTURE]

---

# A/B testing

[ADD PICTURE]

---

# Shadow

[ADD PICTURE]

---

# To be continue...

Scuba Diving Baptism with Istio

^ Introduction to Istio and demo of an app deployment using canary with Istio cross namespaces

---

# Resources

- https://linkerd.io/2/getting-started/
- https://github.com/ContainerSolutions/k8s-deployment-strategies
- https://container-solutions.com/kubernetes-deployment-strategies/
- https://thenewstack.io/deployment-strategies/
- https://medium.com/@cgrant/deployment-strategies-release-best-practices-6e557c3f39b4
- https://rollout.io/blog/deployment-strategy/
- https://cabforward.com/the-difference-between-reliable-and-resilient-software/
- https://docs.deckset.com/English.lproj/Presenting/01-presenter-notes.html
- https://speakerdeck.com/zot24/from-eval-to-prod-how-a-service-mesh-helped-us-build-production-cloud-native-services
