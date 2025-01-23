# Module 5 - Application Level Observability

L7 logs capture application interactions from HTTP header data in requests. Data shows what is actually sent in communications between specific pods, providing more specificity than flow logs. (Flow logs capture data only from connections for workload interactions).

Calico Cloud collects L7 logs by sending the selected traffic through an Envoy proxy.

L7 logs are visible in the Manager UI, service graph, in the HTTP tab.

1. Configure Felix for log data collection

   Enable the Policy Sync API in Felix. For cluster-wide enablement, modify the default FelixConfiguration and set the field policySyncPathPrefix to /var/run/nodeagent.

   ```bash
   kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"policySyncPathPrefix":"/var/run/nodeagent"}}'
   ```

2. Configure the ApplicationLayer resource for L7 logs. Ensure that the collectLogs field is set to Enabled.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: operator.tigera.io/v1
   kind: ApplicationLayer
   metadata:
     name: tigera-secure
   spec:
     logCollection:
       collectLogs: Enabled
       logIntervalSeconds: 5
       logRequestsPerInterval: -1
   EOF
   ```

   This creates l7-log-collector daemonset in calico-system namespace.

   Ensure that the daemonset progresses and l7-collector and envoy-proxy containers inside the daemonset are in a Running state.

3. Select traffic for L7 log collection

   Annotate the frontend service to collect L7 logs as shown.

   ```bash
   kubectl annotate svc facts -n catfacts projectcalico.org/l7-logging=true
   ```

## Service Graph

To view L7 logs in Service Graph:

In the Manager UI left navbar, click Service Graph.

In the bottom pane you will see L7 logs in the HTTP tab.

![http_logs](https://user-images.githubusercontent.com/104035488/216352791-bdbb8376-3b24-4590-81f4-a6b411c1a1cd.gif)

## Dashboards

Calico Cloud provides a set of dashboards to help you understand the activity in your cluster. Each dashboard is made up of graphs, charts, and diagrams that visually represent the data in your logs.

To view your dashboards, sign in to Calico Cloud Manager and click the Dashboards icon.

### L7 HTTP dashboard

The L7 dashboard provides application performance metrics for inscope Kubernetes services. The data can assist service owners and platform personnel in assessing the health of cluster workloads without the need for a full service mesh. L7 logs are not enabled by default, and must be configured.

![l7-dashboard](https://user-images.githubusercontent.com/104035488/216352987-23be3658-2a66-437f-b791-31340971c287.png)

### DNS dashboard

The DNS Dashboard summarizes DNS data and logs into metrics, providing high-level information on the types of DNS lookups made, responses, and overall DNS performance.

![dns-dashboard](https://user-images.githubusercontent.com/104035488/216353120-e78261a0-cd5b-4b89-a171-4225608422c3.png)

---

[:arrow_right: Module 6 - Clean up](module-6-clean-up.md)  

[:arrow_left: Module 4 - Ingress and Egress access control using NetworkSets](module-4-network-sets.md)  
[:leftwards_arrow_with_hook: Back to Main](../README.md)  
