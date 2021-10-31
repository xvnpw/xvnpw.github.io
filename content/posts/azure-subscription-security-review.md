---
title: "Azure subscription security review"
date: 2021-02-01T10:14:47+01:00
draft: false
tags: [azure, devops, appsec]
---

Lately I have come across task to perform security review of Azure subscription. It was white-box based and I had access to all terraform, Kubernetes and Docker files. I will share with you what checks are worth to do for such review.

{{< figure src="https://user-images.githubusercontent.com/17719543/139584399-d5004590-c179-4a87-8c21-aa364f53e293.png" class="center" >}}

## My methodology

Number of resources to check may vary from project to project. For me best approach is to mix automated tools with manual work. If it's possible I'm using two tools for same purpose. It makes my life easier as I don't need to choose best tool 🙂 It's also improving results.

I categorize findings into groups:

* Critical - need to contact team and fix immediately
* High - need to be fixed by team
* Medium - nice to be fixed
* Low - least important to fix

### 0º Scope definition

Before starting any actual work, we need to define scope. In many cases it will directly come from team that created resources in Azure. In that case schedule meeting with them and note all important information:

* what is name of Azure subscription to test (you don't want to review wrong one!)
* what is usage of subscription: e.g. production environment for web application ?
* link to repository with terraform/Kubernetes/Docker files
* any diagrams reflecting how components are communicating, e.g. UML component diagram or network schema
* ask team to go briefly with Azure resources description - no need to be detailed at this point
* what is team priority for review - this is quite important question. On one hand it shows expectations from team, but also can potentially reveal strong and weak sides of configuration.
* establish with team way to contact during review. This time you can give some your expectations, e.g. do status meetings daily and contact immediately on critical findings.

### 1º Turn on Azure Defender

Now it's finally time to go into Azure portal.

At beginning my first steps are into Security Center to turn on paid version of it (currently named Azure Defender). This will do additional scanning and give results of regulatory compliance, e.g. ISO 27001, PCI DSS. Generally speaking this paid version **should be turned on already** at this point as it's giving a lot into cloud security.

Security Center is based in some part on logs coming from agents, so let's give it some time to run.

### 2º Manual check

Having head full of knowledge from scope definition it's time to get familiar with target subscription. This step is based more on intuition than strictly technical. Try to go from resource to resource checking network and security configuration. If something is having "bad smell" just follow it to clarify for 100% whether is good or not. Typically I look for:

* open Storage Accounts
* open ports in Network Security Groups (NSGs)
* not protected Virtual Machines
* misconfiguration in deployed services: databases, Elasticsearch, Redis, etc.
* wrong or to permissive RBAC roles assignment

### 3º Terraform files check

If you are lucky and team is using Infrastructure as a Code, you can test it with automated tools. There are good in finding typical misconfiguration. For terraform I'm using two:

* https://github.com/bridgecrewio/checkov
* https://github.com/tfsec/tfsec

### 4º Kubernetes deployments check

Kubernetes files also can be checked with automated tools. The other way is to check configuration by deploying audit application into cluster. It has benefits as it constantly reporting. It can be running after audit to make team aware about problems.

In my case, I was checking Helm scripts. If you don't know Helm, it's templating language for Kubernetes. In order to check template I needed first to generate Kubernetes files based on template and values. For most simple case you need to run:

```helm template review ./ > output.yaml```

I have choose checkov to test Kubernetes files:

```checkov --quiet -f output.yaml > checkov.result.txt```

Second tool that I recommending is kube-scan deployed into cluster:

```bash
az login
az aks get-credentials -resource-group myResourceGroup -name myAKSCluster
kubectl apply -f https://raw.githubusercontent.com/octarinesec/kube-scan/master/kube-scan-lb.yaml
kubectl -n kube-scan get service kube-scan-ui
```

### 5º Docker check

Dockerfiles can have misconfiguration and docker container can have vulnerabilities, e.g. outdated system packages.

For checking Dockerfiles I used https://github.com/hadolint/hadolint:

```docker run --rm -i hadolint/hadolint < Dockerfile```

and for docker containers https://github.com/anchore/anchore-engine:

{{< gist xvnpw 7e7dda58486e4f661f9b46b51dcbe79c >}}

### 6º Complex security audit tool

Last step in checking is based on tool that is doing complex whole subscription check based on numerous rules, mostly coming from [CIS Benchmarks](https://www.cisecurity.org/benchmark/azure/).

The only tool worth mention that I have found is https://github.com/nccgroup/ScoutSuite. It can check not only Azure but also AWS and GCP.

I had problem in running ScoutSuite on my local machine, so I did run it eventually on Docker. Problem was related to my version of python.

{{< gist xvnpw ec08c001b41f29a7f1bb199150fdb7f3 >}}

### 7º Writing report (if not did yet)

Best way to write report is to do it during review. So that you can take screenshots along the way.

## Summary

I must say that I'm enjoying this kind of tasks in my work 🙂 There are good tools and in most cases running smoothly. Biggest problems I had in playing with `anchor` and `ScoutSuite`.

Findings? In my case there were some critical one. Sadly team didn't spot right recommendations in Azure Security Center on time. Monitoring those recommendations manually is not easy for development teams. Good thing is that Azure is giving some options for notifications.

Please share in comments your ideas about Azure review. What tools are good for you?

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).
