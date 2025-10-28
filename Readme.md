# Chainlink Node Deployment: Setup, Monitoring and Incident Response

## Table of Contents
- [Description](#description)
- [Assumptions](#assumptions)
- [Deploying the Chainlink Node](#deploying-the-chainlink-node)
- [Log collection and Analysis](#log-collection-and-analysis)
- [Monitoring Tools](#monitoring-tools)
- [Handling Incidents](#handling-incidents)

### Description
- Setup, preparing and monitoring a Chainlink node involves several steps from the initial deployment to ensuring that the system is able process jobs efficiently. Below are the instructions on setting up the node, preparing the node, configuring monitoring tools and handling incidents.

### Assumptions:
- You are using a Linux environment (Ubuntu or similar) for the deployment.
- You have Docker and Docker Compose installed.
- You have access to an Ethereum node (e.g., via QuickNode,Infura or your own Ethereum client).
- The goal is to run a Chainlink node 

---

### Deploying the Chainlink Node

#### Step 1: Clone Chainlink Repository

``` yaml
git clone https://github.com/kamal9002/chainlink-node-lab.git
cd chainlink-node-lab
```

#### Step 2: Enviroment Setup

Set up credentails & Environment Variables:
 - Replace __<YOUR_QUICKNODE_ID>__ in config.toml and __PASSWORD__ with your actual values in following files:
    - docker-compose-yml
    - secret.toml

    __<YOUR_QUICKNODE_ID>__ - update httpurl & wssurl for Ethereum Test provider  from (QuickNode, Alchemy,etc)
    Replace the placeholders with your desired email and password `.api` for UI credentails to login in operator

#### Step 3: Deploying the Stack

##### Stack Overview

| Service                     | Description                                                          | Ports  |
| --------------------------- | -------------------------------------------------------------------- | ------ |
| **chainlink_v2-node**       | Chainlink Node service exposing API/UI and job execution             | `6688` |
| **chainlink_v2-postgres**   | PostgreSQL database storing jobs and node data                       | `5432` |
| **loki**                    | Centralized log aggregation service                                  | `3100` |
| **promtail**                | Log collector for Docker containers → sends logs to Loki             | —      |
| **node_exporter**           | Host-level metrics exporter (CPU, memory, disk, network)             | `9100` |
| **prometheus**              | Metrics collection system; scrapes Node Exporter & Chainlink metrics | `9090` |
| **grafana**                 | Visualization & alerting platform; connects to Prometheus & Loki     | `3000` |


After updating environment variables and credentials, navigate to the `docker-compose.yml` directory and run the commands.
                 
```yaml
    docker compose up -d  # Bring up the stack 
    docker compose ps   # check all the containers are healthy and up
```
  ![Docker](images/docker-status.png)

once  all the containers are healthy , access the operator UI in the browser 
 - [Operator UI](http://localhost:6688/)



### Preparing Your Chainlink Node
 - Deploying smart contracts and executing job requests involve multiple steps, all of which are detailed in the Chainlink documentation
     - [Fulfilling Requests Guide](https://docs.chain.link/chainlink-nodes/v1/fulfilling-requests)

 - After completing the above setup, you can verify successful job execution from the Node Operator’s UI
    ![Jobs](images/job_success.png)

### Log collection and Analysis
 - This stack is pre-configured with log aggregation and collector.Once Promtail pushes logs to Loki, you can query and visualize them in grafana
   - Example query:
       Go to _Explore_ in Grafana, select _DS_LOKI_, and run the query **job="chainlink-node"** _(labels according to promtail config)_

    
    ![Logs](images/log-analysis.png)

### Monitoring Tools
 - This stack is pre-configured with infrastructure monitoring. It sets up **Prometheus**, the **Node Exporter** to gather system metrics from the host, and **Grafana** for visualization, with the datasource and dashboard automatically provisioned on startup.
 
    - [grafana ui](http://localhost:3000/) 
    - [Prometheus ui](http://localhost:9090/)
  
    - Node Exporter Dashboard for system metrics

      ![dashboard](images/dashboard.png)

### Handling Incidents
 - step-by-step instructions on managing and resolving incidents, refer to the [playbook-incident](docs/Incident-Response.md)
