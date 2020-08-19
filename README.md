# Sentia case of study

Original premise: https://github.com/sentialabs/public-cloud-recruitment/blob/master/ASSIGNEMENT.md


## Infrastructure Architecture Diagram

Draw<span></span>.io link: https://drive.google.com/file/d/197FaQdYzgOun2gNo18B4MF_6uT1MyUOU/view?usp=sharing

![Infrastructure Architecture Diagram](IAD/Sentia.png)


# Architecture decisions
Infrastructure is a high availability layout balancing traffic to a primary region, but replicating to a secondary region in different fashion for each component. Applications (e.g. Wordpress sites) will be containerized and deployed to Azure Kubernetes Services where we can establish an autoscaling for pods and nodes, and thus adapt to different traffic load.

## Components
- __Resource Group__: Main resource group containing all components. It was considered to create a second one for replicated/balanced components, but it doesn't really bring any benefits (Resource Groups are only logical containers) and it would increase complexity in ARM template deployment. Location for it is primary region.

    _Note: There's another Resource Group created for Azure Monitor services, specifically for Log Analytics workspace. However, this is not part of the main ARM deployment, although there is a template for deploying it beforehand to fill monitoring dependencies. [Link](ARM/LogAnalytics/)_

- __Traffic Manager Profile__: Balances traffic based on priority where primary location is preferred, and secondary location has a lowest priority. It's configured with a CNAME record created in Azure DNS Zone created in the Resource Group. Both backend are treated as external endpoints pointing to two A records also created in the same DNS zone. It's intended for those records to be modified once AKS clusters have their own ingress controller with a public IP address.

- __Azure DNS Zone__: Set up with a default value of `sentiademo.com` it's created so records for the services can be managed in the deployment. During first implementation, it's used by Traffic Manager profile to create endpoints URIs.

- __Storage Account__: Deployed with SKU `Standard_GRS` so it's replicated accross regions. This provides enough reliability in case of a downtime in the primary region. It's intended for serving files to be hosted by the applications, and it's configured to serve them using a CDN to improve user experience.

- __Container Registry__: Registry destined to be consumed by AKS clusters, providing a geo-replication service (primary and secondary location) for redundancy, and closer geographic access to each cluster, but only having one public endpoint which simplifies it's consumption. Geo-replication requires a `Premium` SKU.
