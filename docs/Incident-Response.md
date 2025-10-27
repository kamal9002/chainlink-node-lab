Chainlink Node Incident Response Runbook

**Version:** 1.0

**Last Updated:** 2025-10-28

**Maintained by:** DevOps / Chainlink Reliability Team


## Overview

This runbook defines the standard response procedures for **Chainlink node failures**


## 1. Chainlink Node & PostgreSQL Health Checks Are “Healthy,” But UI Is Unresponsive

### **Detection**
  - Check Node Exporter metrics:
    - CPU, memory, disk usage
    - High load or memory exhaustion may indicate node unresponsiveness
    - node_filesystem_avail_bytes
    - node_network_transmit_bytes_total / _receive_bytes_total
    - Inspect the Reason if any failed containers

      ``` 
        docker container inspect <chainlink_container_name>/<Failed_container_name> 
      ```

      - Check DB Query Health

      ```
        docker exec -it <chainlink_postgres_name>  psql -U chainlink -d chainlink -c \
        "SELECT pid, state, query, now() - query_start AS runtime FROM pg_stat_activity WHERE state!='idle' ORDER BY runtime DESC LIMIT 5;"
        pid | state  |                                                               query                                                               | runtime  
        -----+--------+-----------------------------------------------------------------------------------------------------------------------------------+----------
          96 | active | SELECT pid, state, query, now() - query_start AS runtime FROM pg_stat_activity WHERE state!='idle' ORDER BY runtime DESC LIMIT 5; | 00:00:00
        (1 row)
      ```
      - Check for Log Flooding and Look for repeating errors like pq: canceling statement due to statement timeout , pipeline execution timeout

           ```docker container logs <chainlink_container_name>  | tail -n 100```

      - Test API Responsiveness (Bypass UI)

         ```curl -s -o /dev/null -w "%{http_code}\n" http://localhost:6688/v2/jobs```

        `401` means __reachable but unauthorized__,so the node is alive.

      - Check Resource Usage for all containers 

          ```docker container stats <chainlink_container_name>  <chainlink_postgres_name>```
      
      - ensure the connection between db and app

          ```
            docker container exec -it chainlink_app /bin/bash
            curl chainlink_v2-postgres:5432
            curl: (52) Empty reply from server
          ```
          `curl: (52)` successfully reached the hostname chainlink_v2-postgres on port 5432. The connection was established, but PostgreSQL closed it immediately, since it’s a binary protocol, not HTTP

### **Mitigation & Resolution**

  - Ensure there are no network issues (e.g., DNS resolution failure, firewall rules) 
  - Delete or archive old job/log data
  - Clear unused logs, enlarge DB volume
  - Restart the Node Process

    ```docker compose restart  restart <chainlink_db>```

  - Wait 1–2 minutes and to Confirm container is running

     ```
        docker container logs -f <chainlink_db> 
        sleep 10
        docker compose restart <chainlink>
     ```

  - Check Chainlink UI (port `6688`) is  responsive and normal

  - verify and monitor stability

### **Post-Incident Review**
   - Record the Root Cause Analysis, summarize the resolution and lessons learned, and outline the associated action items.


## 2. Chainlink Node cannot connect to the external Ethereum RPC endpoint (QuickNode) 

### **Detection**
  - Node cannot fetch blockchain data (e.g., jobs fail, balance cannot be read).
  - Chainlink node operations (job runs, data feeds, transactions) may fail or be delayed due to lack of blockchain connectivity.

    Look for:
    - RPC timeout or network unreachable

  ```
    docker container logs <chainlink_container_name> 2>&1 | grep -E "eth_call timeout|connection refused|context deadline exceeded" | tail -n 50

  ```
   - Validate QuickNode Status --> https://status.quicknode.com/

 ### **Mitigation & Resolution**
  - Switch to Backup RPC Endpoint  If available (e.g., Alchemy, Infura, or local Geth node) and restart the container 

      ```docker container restart <chainlink_container_name>```

  - If Chainlink Misconfiguration
     - Verify config.toml contains correct RPC URL and credentials. Restart node and verify connection in logs:

      ```docker container logs -f <chainlink_container_name>```

  - Validate Data Flow
      - Run a test job to ensure node successfully performs `eth_call`


### Post-Incident Review 
  - Record the Root Cause Analysis, summarize the resolution and lessons learned, and outline the associated action items.
