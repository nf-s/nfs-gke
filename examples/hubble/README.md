# GKE Dataplane V2 observability tools
This example requires GKE Dataplane V2 observability feature be enabled. **Enabling this feature will provision an internal network load balancer**.


### Hubble CLI
I suggest making an alias if you already have the Hubble CLI installed on your system, as you will be calling `hubble` from one of the hubble-relay pods:

```console
alias gkehubble='kubectl exec -it -n kube-system deployment/hubble-relay -c hubble-cli -- hubble'

gkehubble status
```

- output:
```
Healthcheck (via localhost:4245): Ok
Current/Max Flows: 63/63 (100.00%)
Flows/s: 9.33
Connected Nodes: 1/1
```


### Hubble UI
```console
kubectl apply -f hubble-ui-auto.yaml
```

```console
kubectl -n kube-system port-forward service/hubble-ui 16100:80 --address='0.0.0.0'
```

Afterwards, you should be able to access the Hubble UI at [http://localhost:16100](http://localhost:16100)


## Cleanup
At the time of writing, this feature is still in [Preview](https://cloud.google.com/products#product-launch-stages) as well as being a recent addition to the Terraform provider.  At the moment, performing a `terraform destroy` will not destroy the network ILB or firewall rules that go with it.  You will have to first disable the DPV2 observability mode before destroying.  If you forgot, it's not a big deal, but you will be met with an error near the end as the VPC network will still have resources attached and all you have to do is manually delete the ILB and the (2) associated firewall rules before rerunning the destroy command.
