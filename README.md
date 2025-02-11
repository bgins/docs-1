---
description: Welcome to the Bacalhau documentation!
icon: hand-wave
cover: .gitbook/assets/bacalhau_banner_high_resolution.png
coverY: 0
---

# Welcome

{% hint style="success" %}
* Bacalhau is now using [NATS](https://nats.io/).&#x20;
* The embedded integration of libp2p and IPFS is deprecated from the version v1.4.0. going forward.
* The API and the job specifications changed from previous versions.

For more information, check out the [release notes](help-and-faq/release-notes.md).
{% endhint %}

## What is Bacalhau?

Bacalhau is a platform for fast, cost efficient, and secure computation by running jobs where the data is generated and stored. With Bacalhau, you can streamline your existing workflows without the need of extensive rewriting by running arbitrary Docker containers and WebAssembly (WASM) images as tasks. This architecture is also referred to as **Compute Over Data** (or CoD). [_Bacalhau_](https://translate.google.com/?sl=pt\&tl=en\&text=bacalhau\&op=translate) _was coined from the Portuguese word for salted Cod fish_.

Bacalhau seeks to transform data processing for large-scale datasets to improve cost and efficiency, and to open up data processing to larger audiences. Our goals is to create an open, collaborative compute ecosystem that enables unparalleled collaboration. We ([Expanso.io](https://expanso.io)) offer a demo network so you can try out jobs without even installing. Give it a shot!

### Why Bacalhau?

{% tabs %}
{% tab title="Data Scientists" %}
* [x] _**Scalability and Flexibility**_:
  * **You** can run large-scale computations without relying on a single cloud provider, enhancing flexibility and potentially reducing costs.
  * Bacalhau enables distributed data processing, which can significantly speed up analysis and model training by parallelizing tasks across multiple nodes.
* [x] _**Reproducibility**_:
  * By using Docker or WASM images, data scientists can ensure that their experiments and models are reproducible across different environments.
* [x] _**Data Privacy and Security**_:
  * Bacalhau allows data to be processed close to its source, which can help maintain data privacy and comply with regulatory requirements.
* [x] _**Cost Efficiency**_:
  * Utilize Bacalhau’s platform to dynamically allocate resources, ensuring optimal performance while controlling costs.
{% endtab %}

{% tab title="DevOps/MLOps" %}
* [x] _**Automation and Orchestration**_:
  * **Seamless Integration**: Bacalhau can be integrated into CI/CD pipelines, enabling automated deployment and scaling of machine learning models and other applications.
  * **Workload Scheduling**: Efficiently schedule and manage workloads across a decentralized network, improving resource utilization and reliability.
* [x] _**Fault Tolerance**_:
  * Decentralized infrastructure ensures high availability and resilience against failures, reducing downtime for critical applications.
* [x] _**Scalable Infrastructure**_:
  * Quickly scale resources up or down based on demand, providing a responsive infrastructure for varying workloads.
* [x] _**Cost Management**_:
  * Utilize Bacalhau’s platform to dynamically allocate resources, ensuring optimal performance while controlling costs.
{% endtab %}

{% tab title="IT Operations" %}
* [x] _**Infrastructure Efficiency**_:&#x20;
  * Efficiently utilize idle or underutilized compute resources within an organization, maximizing hardware investments.
* [x] _**Simplified Management**_:&#x20;
  * Manage heterogeneous compute resources through a single platform, simplifying administrative tasks and reducing complexity.
* [x] _**Enhanced Security**_:&#x20;
  * Reduce the risk of centralized points of failure and potential security breaches by leveraging a decentralized network for data processing.
* [x] _**Cost Reduction**_:&#x20;
  * Bacalhau’s can help drive down your compute costs by up to 72.5% for deploying your ML models and over 90% for your log processing spend.
* [x] _**Adaptable Infrastructure**_:&#x20;
  * Easily adapt to changing business requirements by scaling infrastructure resources as needed without significant upfront investment.
{% endtab %}
{% endtabs %}

{% hint style="info" %}
⚡️ Bacalhau simplifies the process of managing compute jobs by providing a **unified platform** for managing jobs across different regions, clouds, and edge devices.
{% endhint %}

### How it works

Bacalhau consists of a network of nodes that enables orchestration between every compute resource, no matter whether it is a Cloud VM, an On-premise server or Edge devices. The network consists of two types of nodes:

**Requester Node:** responsible for handling user requests, discovering and ranking compute nodes, forwarding jobs to compute nodes, and monitoring the job lifecycle.

**Compute Node:** responsible for executing jobs and producing results. Different compute nodes can be used for different types of jobs, depending on their capabilities and resources.

{% hint style="info" %}
For a more detailed tutorial, check out our [Getting Started Tutorial](broken-reference).
{% endhint %}

#### Data ingestion

Data is identified by its content identifier (CID) and can be accessed by anyone who knows the CID. Here are some options that can help you mount your data:

* [Copy data from a URL to public storage](setting-up/data-ingestion/from-url.md)
* [Pin Data to public storage](setting-up/data-ingestion/pin.md)
* [Copy Data from S3 Bucket to public storage](setting-up/data-ingestion/s3.md)

{% hint style="info" %}
The options are not limited to the above mentioned. You can mount your data anywhere on your machine, and Bacalhau will be able to run against that data
{% endhint %}

#### Security in Bacalhau

All workloads run under restricted Docker or WASM permissions on the node. Additionally, you can use existing (locked down) binaries that are pre-installed through Pluggable Executors.

Best practices in [12-factor apps](https://12factor.net/) is to use environment variables to store sensitive data such as access tokens, API keys, or passwords. These variables can be accessed by Bacalhau at runtime and are not visible to anyone who has access to the code or the server. Alternatively, you can pre-provision credentials to the nodes and access those on a node by node basis.

Finally, endpoints (such as vaults) can also be used to provide secure access to Bacalhau. This way, the client can authenticate with Bacalhau using the token without exposing their credentials.

### Use Cases

Bacalhau can be used for a variety of data processing workloads, including machine learning, data analytics, and scientific computing. It is well-suited for workloads that require processing large amounts of data in a distributed and parallelized manner.

Once you have more than 10 devices generating or storing around 100GB of data, you're likely to face challenges with processing that data efficiently. Traditional computing approaches may struggle to handle such large volumes, and that's where distributed computing solutions like Bacalhau can be extremely useful. Bacalhau can be used in various industries, including security, web serving, financial services, IoT, Edge, Fog, and multi-cloud. Bacalhau shines when it comes to data-intensive applications like [data engineering](examples/data-engineering/), [model training](examples/model-training/), [model inference](examples/model-inference/), [molecular dynamics](examples/molecular-dynamics/), etc.

{% embed url="https://www.youtube.com/watch?ab_channel=Expanso&t=7s&v=R5s9cZg5DOM" %}
An example on how to build your own ETL pipeline with Bacalhau and MongoDB.
{% endembed %}

Here are some example tutorials on how you can process your data with Bacalhau:

* [Stable Diffusion AI](broken-reference) training and deployment with Bacalhau.
* [Generate Realistic Images using StyleGAN3 and Bacalhau](broken-reference).
* [Object Detection with YOLOv5 on Bacalhau](broken-reference).
* [Running Genomics on Bacalhau](broken-reference).
* [Training Pytorch Model with Bacalhau](broken-reference).

{% hint style="info" %}
For more tutorials, visit our [example page](broken-reference)
{% endhint %}

### Community

Bacalhau has a very friendly community and we are always happy to help you get started:

* [GitHub Discussions](https://github.com/bacalhau-project/bacalhau/discussions) – ask anything about the project, give feedback or answer questions that will help other users.
* [Join the Slack Community](https://bit.ly/bacalhau-project-slack) and go to **#bacalhau** channel – it is the easiest way engage with other members in the community and get help.
* [Contributing](community/ways-to-contribute.md) – learn how to contribute to the Bacalhau project.

### Next Steps

👉 Continue with Bacalhau [Getting Started guide](broken-reference) to learn how to install and run a job with the Bacalhau client.

👉 Or jump directly to try out the different [Examples](broken-reference) that showcases Bacalhau abilities.
