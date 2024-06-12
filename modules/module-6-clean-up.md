# Module 8 - Clean up

1. Delete the applications stack to clean up any `loadbalancer` services.

   ```bash
   kubectl delete -f pre/40-catfacts-app.yaml
   ```

2. Delete EKS cluster.

   ```bash
   source ~/workshopvars.env
   eksctl delete cluster --name $CLUSTERNAME --region $REGION
   ```

3. Delete this repo

   ```bash
   cd .. && rm -Rf cc-eks-zero-trust-workshop
   ```

4. Delete environment variables backup file.

   ```bash
   rm ~/workshopvars.env
   ```

---

[:arrow_left: Module 5 - Application Level Observability](module-5-application-observability.md)  
[:leftwards_arrow_with_hook: Back to Main](../README.md)  
