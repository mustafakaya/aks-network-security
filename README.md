# Azure Kubernetes Service : Network Security Recommendations

Network security recommendations focus on specifying which network protocols, TCP/UDP ports, and network connected services are allowed or denied access to Azure services. We have to ensure that all Virtual Network subnet deployments have a Network Security Group applied with network access controls specific to your application's trusted ports and sources. 

We have two network options for AKS cluster network which are kubenet and Azure CNI. Kubenet is default and simple network plugin for AKS clusters and typically used together with a cloud provider. With kubenet, nodes get an IP address from the Azure virtual network subnet and pods receive an IP address from a logically different address space.  With Azure CNI, every pods get an IP address from the subnet and can be accessed directly.  Kubenet is basic networking and Azure CNI is advanced networking. 

With Kubenet, pods can't communicate directly with each other. Instead, User Defined Routing (UDR) and IP forwarding is used for connectivity between pods across nodes. By default, UDRs and IP forwarding configuration is created and maintained by the AKS service, also you have to the option to bring your own route table for custom route management. 

There's an important point: using extra routing with UDR and IP forwarding in Kubenet may reduce network performance. 


![image](https://user-images.githubusercontent.com/9195953/111771755-1fe17d80-88bd-11eb-8fae-8a122b27a8d5.png)  ![image](https://user-images.githubusercontent.com/9195953/111771795-2a037c00-88bd-11eb-9ab4-cd74263149b5.png)
	
 


## 1- Use Firewall

The AKS outbound dependencies are almost entirely defined with FQDNs, which don't have static addresses behind them. The lack of static addresses means that Network Security Groups can't be used to lock down the outbound traffic from an AKS cluster.

If you wish to restrict egress traffic, a limited number of ports and addresses must be accessible to maintain healthy cluster maintenance tasks. The simplest solution to securing outbound addresses lies in use of a firewall device that can control outbound traffic based on domain names. Azure Firewall, for example, can restrict outbound HTTP and HTTPS traffic based on the FQDN of the destination. You can also configure your preferred firewall and security rules to allow these required ports and addresses.

## 	2- Use AKS Network Policies

Network security group and route table automatically created with AKS cluster and AKS automatically modifies network security groups. But network security groups can't be used to lock down the outbound traffic from an AKS cluster and blocking internal subnet traffic.  

All traffic is allowed between pods within a cluster, we should implement a cloud native way to automatically handle traffic when new pods created.  And blocking internal subnet traffic using network security groups and firewalls is not supported. 

We should use AKS Network Policies to control and block network traffic by defining rules for ingress and egress traffic between Linux pods in a cluster . By default all pods can communicate each other in an AKS cluster and we can use network policies to control traffic in cluster.  

Azure provides two ways to implement network policy:

	- Azure's own implementation, called Azure Network Policies (support only Azure CNI network plugin, supported by Azure support and engineering team)
	- Calico Network Policies, an open-source network and network security solution founded by Tigera (support Azure CNI and Kubenet network plugin)

Important note: We can't enable network policy on an existing AKS cluster. We have to enable while creating an AKS cluster.

## 	3- Use Private AKS Cluster

By default, the Kubernetes API server uses a public IP address and a fully qualified domain name (FQDN). Private AKS cluster ensures network traffic between AKS API server and node pools remains on the private network only. The control plane (or API Server) is in an AKS managed Azure subscription, node pool is in the customer managed subscription. The control plane and node pool can communicate each other through the Azure Private Link service in the API server virtual network and a private endpoint that's exposed in the subnet of the customer's AKS cluster.

## 	4- Use Web Application Firewall (WAF) 

We use ingress controller to distribute HTTP/S traffic to our applications and services which are resources in AKS cluster. And we can scan incoming traffic for potential attacks with Web Application Firewall which provides an extra layer to filter the traffic. 

Azure Application Gateway is a WAF that can integrate with AKS clusters to provide OWASP rules as security features before traffic reaches AKS cluster. Also App Gateway can be managed as an ingress controller. 

There're other third party solutions provides same functions such as Barracuda WAF on Azure .

![image](https://user-images.githubusercontent.com/9195953/111772456-ecebb980-88bd-11eb-8be3-f7c2e2914c82.png)



## 	5- Use API Gateway inside the cluster Virtual Network

AKS helps us to deploy and operate our microservices in the cloud and each microservice expose a set of endpoints. And API gateway servers as a front door to the microservices and adds an additional layers of security. 

We can use an API gateway for authentication, authorization, throttling, caching, transformation, and monitoring for APIs used in our AKS environment. 

Most secure design is deploy API Management inside the cluster Virtual Network. In this deployment model, there's not any service exposed as public endpoint so all API traffic will remain within the virtual network.  That looks complex but that's a most secure option and if you want, you can hide both API Management and AKS inside virtual network using the Internal Mode.

![image](https://user-images.githubusercontent.com/9195953/111772468-f2490400-88bd-11eb-8b0e-3b81a6c45151.png)

![image](https://user-images.githubusercontent.com/9195953/111772617-1b699480-88be-11eb-8d6e-18f78425753e.png)




## 	6- Enable DDoS Standard Protection

Distributed denial of service (DDoS) attacks are some of the largest availability and security concerns facing customers that are moving their applications to the cloud. Every property in Azure is protected by Azure's infrastructure DDoS (Basic) Protection at no additional cost. The scale and capacity of the globally deployed Azure network provides defense against common network-layer attacks through always-on traffic monitoring and real-time mitigation. DDoS Protection Basic requires no user configuration or application changes. DDoS Protection Basic helps protect all Azure services, including PaaS services like Azure DNS.

We should enable DDoS protection on the virtual networks where AKS components are deployed of protection against DDoS attacks.

## 	7- Enable Network Watcher to Record Network Packets

Network Watcher is a regional service that enables you to monitor and diagnose conditions at a network scenario level in, to, and from Azure. Scenario level monitoring enables you to diagnose problems at an end to end network level view. Network diagnostic and visualization tools available with Network Watcher help you understand, diagnose, and gain insights to your network in Azure. 

Network Watcher enabled automatically in virtual network's region when we create or update a virtual network in subscription.

We should use Network Watcher packet capture to investigate anomalous activity.

## 	8- Use Azure Policy to Define and Implement Standard Security Configurations

Azure Policy helps us to enforce organizational standards to our resources. We should use Azure Policy to audit and enforce the network configuration of our AKS clusters.

We can create custom policies using "Microsoft.ContainerService" and "Microsoft.Network" namespaces or we can use built-in policy definitions related to AKS, such as:

	- Authorized IP ranges should be defined on Kubernetes Services
	- Enforce HTTPS ingress in Kubernetes cluster
	- Ensure services listen only on allowed ports in Kubernetes cluster
	- Kubernetes cluster services should only use allowed external IPs
	- Kubernetes cluster pods should only use approved host network and port range
	- Kubernetes clusters should be accessible only over HTTPS

## 	9- Use Activity Log and Azure Monitor to Monitor Network Resources

Activity Log helps us to monitor network resource configurations and detect changes for network resources related to AKS clusters. Create alerts within Azure Monitor that will trigger when changes to critical network resources place take. 

Also Azure Monitor collect logs from AKS components such as control-plane, api-server and kube-controller-manager which help us investigate the issues. 


References:

- http://docs.kubernetes.io/
- http://docs.microsoft.com/en-US/azure/aks

