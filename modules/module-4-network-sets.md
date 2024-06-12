# Module 4 - Ingress and Egress access control using NetworkSets

It's also possible to create network sets for controlling access from and to a  list of domain names and ip addresses. Let's verify this control in action.

1. Test the access to different APISs from the endpoint `catfacts/worker`:

   ```bash
   # test egress access to catfact.ninja from db pod
   kubectl -n catfacts exec -t $(kubectl -n catfacts get po -l app=worker -ojsonpath='{.items[0].metadata.name}') -- sh -c 'curl -m3 -skI https://catfact.ninja/fact 2>/dev/null | grep -i http'
   ```

   ```bash
   # test egress access to dog.ceo from db pod
   kubectl -n catfacts exec -t $(kubectl -n catfacts get po -l app=worker -ojsonpath='{.items[0].metadata.name}') -- sh -c 'curl -m3 -skI https://dog.ceo/api/breeds/image/random 2>/dev/null | grep -i http'
   ```

   ```bash
   # test egress access to api.twilio.com
   kubectl -n catfacts exec -t $(kubectl -n catfacts get po -l app=worker -ojsonpath='{.items[0].metadata.name}') -- sh -c 'curl -m3 -skI https://api.twilio.com 2>/dev/null | grep HTTP'
   ```

   Access to the `api.twilio.com` endpoint should be denied by the DNS policy.

What if we now need access to the Twilio API? We would need to add the domain api.twilio.com to each DNS policy we have. For our workshop, we only have one policy, but in a real-world scenario, we would likely have dozens of workloads accessing different APIs. In that case, we would need to modify each network policy to include the Twilio API.

To simplify this process, we can use a NetworkSet to create a list of URLs or IP addresses and have the network policies refer to it. This way, only the NetworkSet needs to be modified when adding or removing addresses.

Let's test it out.

1. Edit the policy to use a `NetworkSet` with DNS domain instead of inline DNS rule.

   a. Create a NetworkSet that includes the `dog.ceo`, `catfact.ninja` and `api.twilio.com` APIs as allowed egress domains.

   Deploy the NetworkSet

   ```yaml
   kubectl apply -f - <<-EOF
   kind: GlobalNetworkSet
   apiVersion: projectcalico.org/v3
   metadata:
     name: allowed-api
     labels: 
       type: allowed-api
   spec:
     allowedEgressDomains:
     - 'dog.ceo'
     - 'catfact.ninja'
     - 'api.twilio.com'
   EOF
   ```

   b. Deploy the DNS policy using the NetworkSet that we created.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.external-api-access
   spec:
     tier: security
     selector: (app == "worker" && projectcalico.org/namespace == "catfacts")
     order: 100
     types:
       - Egress
     egress:
     - action: Allow
       destination:
         selector: type == "allowed-api"
   EOF
   ```

   c. Test the access to the endpoints.

   ```bash
   # test egress access to catfact.ninja from db pod
   kubectl -n catfacts exec -t $(kubectl -n catfacts get po -l app=worker -ojsonpath='{.items[0].metadata.name}') -- sh -c 'curl -m3 -skI https://catfact.ninja/fact 2>/dev/null | grep -i http'
   ```

   ```bash
   # test egress access to dog.ceo from db pod
   kubectl -n catfacts exec -t $(kubectl -n catfacts get po -l app=worker -ojsonpath='{.items[0].metadata.name}') -- sh -c 'curl -m3 -skI https://dog.ceo/api/breeds/image/random 2>/dev/null | grep -i http'
   ```

   ```bash
   # test egress access to api.twilio.com
   kubectl -n catfacts exec -t $(kubectl -n catfacts get po -l app=worker -ojsonpath='{.items[0].metadata.name}') -- sh -c 'curl -m3 -skI https://api.twilio.com 2>/dev/null | grep HTTP'
   ```

## Ingress Policies using NetworkSets

The NetworkSet can also be used to block access from a specific ip address or cidr to an endpoint in your cluster. To demonstrate it, we are going to block the access from your workstation to the ```facts``` external ```LoadBalancer``` service.

   a. Test the access to the ```facts``` external service

   ```bash
   curl -sI -m3 $(kubectl get svc -n catfacts facts -ojsonpath='{.status.loadBalancer.ingress[0].hostname}') | grep -i http
   ```

   b. Identify your workstation ip address and store it in a environment variable

   ```bash
   export MY_IP=$(curl ifconfig.me)
   ```

   c. Create a NetworkSet with your ip address on it.

   ```yaml
   kubectl apply -f - <<-EOF
   kind: GlobalNetworkSet
   apiVersion: projectcalico.org/v3
   metadata:
     name: ip-address-list
     labels: 
       type: blocked-ips
   spec:
     nets:
     - $MY_IP/32
   EOF
   ```

   d. Create the policy to deny access to the ```facts``` service.

   ```yaml
   kubectl apply -f - <<-EOF
   apiVersion: projectcalico.org/v3
   kind: GlobalNetworkPolicy
   metadata:
     name: security.blockep-ips
   spec:
     tier: security
     selector: app == "facts"
     order: 300
     types:
       - Ingress
     ingress:
     - action: Deny
       source:
         selector: type == "blocked-ips"
       destination: {}
     - action: Pass
       source: {}
       destination: {}
   EOF
   ```

   e. Create a global alert for the blocked attempt from the ip-address-list to the frontend.

   ```yaml
   kubectl apply -f - <<-EOF   
   apiVersion: projectcalico.org/v3
   kind: GlobalAlert
   metadata:
     name: blocked-ips
   spec:
     description: "A connection attempt from a blocked ip address just happened."
     summary: "[blocked-ip] ${source_ip} from ${source_name_aggr} networkset attempted to access ${dest_namespace}/${dest_name_aggr}"
     severity: 100
     dataSet: flows
     period: 1m
     lookback: 1m
     query: '(source_name = "ip-address-list")'
     aggregateBy: [dest_namespace, dest_name_aggr, source_name_aggr, source_ip]
     field: num_flows
     metric: sum
     condition: gt
     threshold: 0
   EOF
   ```

   a. Test the access to the ```facts``` service. It is blocked now. Wait a few minutes and check the `Activity > Alerts`.

   ```bash
   curl -m3 $(kubectl get svc -n catfacts facts -ojsonpath='{.status.loadBalancer.ingress[0].hostname}')
   ```

---

[:arrow_right: Module 5 - Application Level Observability](module-5-application-observability.md)  

[:arrow_left: Module 3 - Workload Isolation with Microsegmentation](module-3-wkload-isolation.md)  
[:leftwards_arrow_with_hook: Back to Main](../README.md)  
