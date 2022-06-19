# Kubernetes Microservice on AWS with Amazon Elastic Kubernetes Service

This example uses a microservice generated with JHipster 7. It includes four applications: a JHipster registry, an application gateway, a store resource server, and a blog resource server. Take a look at [this blog post](TODO) for more information.

## Prerequisites

Install the required software listed below. You’ll need to sign up for an **Amazon Web Services account** and a free **Okta account** (which you can sign up for using the Okta CLI). 

The Kubernetes cluster required to finish this tutorial **does not qualify for free tier** on AWS. However, if you are careful about stopping and deleting the cluster when not actively working on the tutorial, the cost should be very small, only a few dollars.

- Docker: you’ll need to have both Docker Engine and Docker Compose installed (If you install the Docker desktop, this will automatically install both. On Linux, if you install Docker Engine individually, you will have to also install Docker Compose) separately.
- Docker Hub: you’ll need a Docker Hub to host the docker images so that Azure can pull them.
- Java 11: this tutorial requires Java 11. If you need to manage multiple Java versions, SDKMAN! is a good solution. - Okta CLI: you’ll use Okta to add security to the microservice network. You can register for a free account from the CLI.
- Amazon Web Services: sign up for an AWS account
- kubectl: CLI to manage Kubernetes clusters. As of the time I was writing this tutorial, there was a bug in kubectl version 1.24 that results in an “invalid apiVersion” error. You can avoid this error by installing version 1.23.6.

**Table of Contents**

* [Clone the Project](#clone-the-project)
* [Configure Okta OAuth](#configure-okta-oauth)
* [Build Docker images](#build-docker-images)
* [Create EKS cluster](#create-eks-cluster)
* [Deploy microservice to EKS](#deploy-microservice-to-eks)
* [Cleaning Up](#cleaning-up)
* [Links](#links)
* [Help](#help)
* [License](#license)


## Clone the Project

To install this example, run the following commands:

```bash
git clone TODO
cd TODO
```


## Configure Okta OAuth

Open a bash shell and navigate to the project root.

If you already have an Okta account, use okta login to log into that account with the CLI. Otherwise, use okta register to sign up for a free account.

Create the OIDC app using the following command using a Bash shell opened to the project root.

okta apps create jhipster

You can accept the default values by pressing enter. This creates `.okta.env.`

Use the values from .okta.env to fill in the issuer-uri, client-id, and client-secret in the central server config application.yml file.

`docker-compose/central-server-config/application.yml`
```yaml
spring:
  security:
    oauth2:
      client:
        provider:
          oidc:
            issuer-uri: <your-issuer-uri>
        registration:
          oidc:
            client-id: <your-client-id>
            client-secret: "<your-encrypted-key>"
 ```
 
 ## Build Docker images
 
In the `kubernetes` subdirectory, **peform a multifile search and replace**, replacing `<your-docker-repository-name>` with your actual Docker Hub repository name.
 
 Make sure you are logged into docker: `docker login`
 
 The commands to build the Docker images and push them to Docker Hub are the following. You must replace {your-docker-repository-name} with your actual docker repository name and run these commands in the correct project subdirectories.

```bash
# in ./gateway
./gradlew bootJar -Pprod jib -Djib.to.image={your-docker-repository-name}/gateway
# in ./blog
./gradlew bootJar -Pprod jib -Djib.to.image={your-docker-repository-name}/blog
# in ./store
./gradlew bootJar -Pprod jib -Djib.to.image={your-docker-repository-name}/store
```

## Create EKS cluster


Use the following command to create the Kubernetes cluster for the microservice.

```bash
eksctl create cluster --name okta-k8s \
--region us-west-2 \
--nodegroup-name okta-k8s-nodes \
--node-type t2.xlarge \
--nodes 2
```

This will take several minutes to run.

**This cluster costs money as long as it is running.** If you only keep it running for the hour or so you need to run the tutorial, the costs are small. If you forget and leave it running for a month or two, it could cost a lot more. Don’t forget to delete it when you are done!

If you get an error, you may have to specify specific availability zones.
```bash
--zones=us-west-2a,us-west-2b,us-west-2c
```

 ## Deploy microservice to EKS
 
Run the following command to update the AWS kubernetes configuration.

```bash
aws eks update-kubeconfig --name okta-k8s
```

From the `kubernetes` subdirectory, run the following command to deploy the microservice.

```bash
./kubectl -f
```

Check the deployment with:
```bash
kubectl get pods -n demo
```
To access the Eureka registry service, you need to forward the port.

```bash
kubectl port-forward svc/jhipster-registry -n demo 8761
```

Open a browser and navigate to [http://localhost:8761](http://localhost:8761)

Access the gateway application by forwarding its port.
```bash
kubectl port-forward svc/gateway -n demo 8080
```
Open a browser and navigate to [http://localhost:8080](http://localhost:8080)

## Cleaning Up

Once you’re done, **delete the EKS cluster**. If you don’t do this, you’ll keep getting charged for the AWS resources.

 ```bash
 eksctl delete cluster --region=us-west-2 --name=okta-k8s
 ```
 You may also want to log into the AWS console and make sure all of the associated resources have been deleted.
 
 
## Links

This example uses the following open source libraries:

* [Spring Boot](https://spring.io/projects/spring-boot)
* [Spring Cloud](https://spring.io/projects/spring-cloud)
* [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
* [Spring Security](https://spring.io/projects/spring-security)
* [JHipster](https://www.jhipster.tech)
* [OpenJDK](https://openjdk.java.net/)


## Help

Please post any questions as comments on [this example's blog post][blog], or on the [Okta Developer Forums](https://devforum.okta.com/).

## License

Apache 2.0, see [LICENSE](LICENSE).

[blog]: TODO
 
