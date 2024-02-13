# ocp_netobserv role

This role allows enabling the OCP Network Observability subsystem. The main components of the stack are:
  - [Loki](https://grafana.com/oss/loki/) - for logs storage in an Object Storage
  - The flow collector - for collecting network flows
  - Object storage bucket - the permanent logs storage
  - Physical volumes - local storage used for data cache

All the components are created in the `netobserv` namespace.

Main role tasks:
  - Validates the Netobserv subsystem requirements
  - Deploys Loki stack
  - Creates a FlowCollector instance
  - Enabled the console plugin

# Role variables

 Variable                                  | Default                         | Required    | Description                                   
 ----------------------------------------- | ------------------------------- | ----------- | ----------------------------------------------
ocp_netobserv_conf_file                    |                                 | Yes         | Configuration file 


## Configuration Variables

 Variable                              | Default                         | Required    | Description
-------------------------------------- | ------------------------------- | ----------- | ----------------------------------------------
netobserv_action                       | 'install'                       | No          | Role's default action
netobserv_sampling                     | 50                              | No          | Data sampling
netobserv_agent_privileged             | true                            | No          | Privileged mode allows collecting data from SRIOV functions
netobserv_agent_memory                 | 50Mi                            | No          | Memory assigned to the agent
netobserv_agent_cpu                    | 100m                            | No          | CPU assigned to the agent
netobserv_agent_limits_memory          | 800Mi                           | No          | Memory limit for the agent
netobserv_processor_memory             | 100Mi                           | No          | Memory assigned to the processor
netobserv_processor_cpu                | 100m                            | No          | CPU assigned to the processor
netobserv_processor_limits_memory      | 800Mi                           | No          | CPU limit for the processor
netobserv_console_avg_utilization      | 50                              | No          | Average utilization for the console
netobserv_console_max_replicas         | 3                               | No          | Console replicas
netobserv_loki_tls_insecure_skip_verify| true                            | No          | Skip TLS verification
netobserv_access_key_id                | minioadmin                      | No          | Access Key ID for the object storage backend
netobserv_access_key_secret            | minioadmin                      | No          | Secret Key for the object storage backend
netobserv_bucket                       | network                         | No          | Bucket for the Network Observability
netobserv_endpoint                     | http://minio-service.minio:9000 | No          | Object Storage Endpoint. It must exist and be reachable
netobserv_region                       | us-east-1                       | No          | Object Storage region
netobserv_loki_size                    | 1x.extra-small                  | No          | Loki Stack size See [Sizing](https://docs.openshift.com/container-platform/4.13/logging/cluster-logging-loki.html#deployment-sizing_cluster-logging-loki)
netobserv_storage_class                | managed-nfs-storage             | No          | Storage class for the Loki Stack

## Role requirements
  - An Storage Class
  - Loki-operator
  - Network observability operator
  - Supported Log Store (AWS S3, Google Cloud Storage, Azure, Swift, Minio, OpenShift Data Foundation) credentials (access_key and access_key_secret)
  - Logs Store endpoint address

## Usage example

See below for some examples of how to use the ocp_netobserv role to configure the Network Observability subsystem.

Create the Netobserv stack with default values:

```yaml
- name: "Setup Netobserv stack"
  vars:
    ocp_netobserv_conf_file: path-to/netobserv-config.yml
  ansible.builtin.include_role:
    name: redhatci.ocp.ocp_netobserv
```

Use a config file
```yaml
- name: "Setup Netobserv stack"
  vars:
    ocp_netobserv_conf_file: path-to/netobserv-config.yml
  ansible.builtin.include_role:
    name: redhatci.ocp.ocp_netobserv
```

Sample of a configuration file:
```yaml
---
netobserv_action: 'install'
netobserv_sampling: 50
netobserv_agent_privileged: true
netobserv_agent_memory: 50Mi
netobserv_agent_cpu: 100m
netobserv_agent_limits_memory: 800Mi
netobserv_processor_memory: 100Mi
netobserv_processor_cpu: 100m
netobserv_processor_limits_memory: 800Mi
netobserv_console_avg_utilization: 50
netobserv_console_max_replicas: 3
netobserv_console_limits_memory: 100Mi
netobserv_console_cpu: 100m
netobserv_console_memory: 50Mi
netobserv_loki_tls_insecure_skip_verify: true
netobserv_access_key_id: minioadmin
netobserv_access_key_secret: minioadmin
netobserv_bucket: network
netobserv_endpoint: http://minio-service.minio:9000
netobserv_region: us-east-1
netobserv_loki_size: 1x.extra-small
netobserv_storage_class: "managed-nfs-storage"
...

## Validaton

To confirm that the stack is working properly:
1. Validate that a "Logs" entry has been added to the OCP console under the "Observe" menu.
1. Verify that the Object Storage bucket has started receiving files.

## Troubleshooting

The following are some recommended actions in case there are issues during the deployment.

1. Confirm that all the pods in the `netobserv` namespace are running.
1. Confirm that the OCP cluster is able to reach the Object Storage bucket. Check the ingester pods for errors when flushing logs tables.
    ```ShellSession
    $ oc logs -n netboserv logging-loki-ingester-0 | grep flush
    ```

# References

* [Network Observatility Operator documentation](https://docs.openshift.com/container-platform/4.14/network_observability/configuring-operator.html)
* [olm_operator](../olm_operator/README.md): A role that installs OLM based operators.
* [dci-openshfit-agent](https://github.com/redhat-cip/dci-openshift-agent/): An agent that allows the deployment of OCP clusters, it is integrated with DCI (Red Hat Distributed CI).
* [dci-openshfit-app-agent](https://github.com/redhat-cip/dci-openshift-app-agent/): An agent that allows the deployment of workloads and certification testing on top OCP clusters, it is integrated with DCI (Red Hat Distributed CI).
