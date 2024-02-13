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
------------------------------------------ | ------------------------------- | ----------- | ----------------------------------------------
ocp_netobserv_conf_file                    |                                 | No          | Configuration file, this overrides the vars passed to the role
ocp_netobserv_action                       | 'install'                       | No          | Role's default action
ocp_netobserv_sampling                     | 50                              | No          | Data sampling
ocp_netobserv_agent_privileged             | true                            | No          | Privileged mode allows collecting data from SRIOV functions
ocp_netobserv_agent_memory                 | 50Mi                            | No          | Memory assigned to the agent
ocp_netobserv_agent_cpu                    | 100m                            | No          | CPU assigned to the agent
ocp_netobserv_agent_limits_memory          | 800Mi                           | No          | Memory limit for the agent
ocp_netobserv_processor_memory             | 100Mi                           | No          | Memory assigned to the processor
ocp_netobserv_processor_cpu                | 100m                            | No          | CPU assigned to the processor
ocp_netobserv_processor_limits_memory      | 800Mi                           | No          | CPU limit for the processor
ocp_netobserv_console_avg_utilization      | 50                              | No          | Average utilization for the console
ocp_netobserv_console_max_replicas         | 3                               | No          | Console replicas
ocp_netobserv_loki_tls_insecure_skip_verify| true                            | No          | Skip TLS verification
ocp_netobserv_access_key_id                | minioadmin                      | No          | Access Key ID for the object storage backend
ocp_netobserv_access_key_secret            | minioadmin                      | No          | Secret Key for the object storage backend
ocp_netobserv_bucket                       | network                         | No          | Bucket for the Network Observability
ocp_netobserv_endpoint                     | http://minio-service.minio:9000 | No          | Object Storage Endpoint. It must exist and be reachable
ocp_netobserv_region                       | us-east-1                       | No          | Object Storage region
ocp_netobserv_loki_size                    | 1x.extra-small                  | No          | Loki Stack size See [Sizing](https://docs.openshift.com/container-platform/4.14/logging/log_storage/installing-log-storage.html)
ocp_netobserv_storage_class                | managed-nfs-storage             | No          | Storage class for the Loki Stack

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
    ocp_netobserv_sampling: 60
    ocp_netobserv_loki_size: 1x.small
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
ocp_netobserv_action: 'install'
ocp_netobserv_sampling: 50
ocp_netobserv_agent_privileged: true
ocp_netobserv_agent_memory: 50Mi
ocp_netobserv_agent_cpu: 100m
ocp_netobserv_agent_limits_memory: 800Mi
ocp_netobserv_processor_memory: 100Mi
ocp_netobserv_processor_cpu: 100m
ocp_netobserv_processor_limits_memory: 800Mi
ocp_netobserv_console_avg_utilization: 50
ocp_netobserv_console_max_replicas: 3
ocp_netobserv_console_limits_memory: 100Mi
ocp_netobserv_console_cpu: 100m
ocp_netobserv_console_memory: 50Mi
ocp_netobserv_loki_tls_insecure_skip_verify: true
ocp_netobserv_access_key_id: minioadmin
ocp_netobserv_access_key_secret: minioadmin
ocp_netobserv_bucket: network
ocp_netobserv_endpoint: http://minio-service.minio:9000
ocp_netobserv_region: us-east-1
ocp_netobserv_loki_size: 1x.extra-small
ocp_netobserv_storage_class: "managed-nfs-storage"
...

## Access the NetObserv GUI

Once the deployment is complete, there should be a new entry for `Network` under the Observe menu in the main OCP console.

## Troubleshooting

The following are some recommended actions in case there are issues during the deployment.

1. Confirm that all the pods in the `netobserv` namespace are running.
1. Confirm that the OCP cluster is able to reach the Object Storage bucket. Check the ingester pods for errors when flushing logs tables.
    ```ShellSession
    $ oc logs -n netboserv logging-loki-ingester-0 | grep flush
    ```
1. Delete the `flowcollector` resource and the `netboserv` namespace and try re-deploy.
    ```ShellSession
    $ oc delete flowcollector cluster
    $ oc delete ns netboserv
    ```

# References

* [Network Observability Operator documentation](https://docs.openshift.com/container-platform/4.14/network_observability/configuring-operator.html)
* [olm_operator](../olm_operator/README.md): A role that installs OLM based operators.
* [dci-openshfit-agent](https://github.com/redhat-cip/dci-openshift-agent/): An agent that allows the deployment of OCP clusters, it is integrated with DCI (Red Hat Distributed CI).
* [dci-openshfit-app-agent](https://github.com/redhat-cip/dci-openshift-app-agent/): An agent that allows the deployment of workloads and certification testing on top OCP clusters, it is integrated with DCI (Red Hat Distributed CI).
