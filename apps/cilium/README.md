# Why Cilium
Cilium is used for container networking for Hybrid nodes. It provides advanced networking features - BGP routing,
 networking policies and observability for containers running in on-premise htbrid nodes which are part of the AWS EKS clusters.

 # CNI Architecture
 - For regular EC2 nodes those are managed by AWS **AWS VPC CNI**
 - For hybrid nodes - **Cilium CNI**
 - Cilium is configured with BGP[BGP, or Border Gateway Protocol, is the routing protocol used on the internet to exchange routing and reachability information between different autonomous systems (AS), which are large networks controlled by a single organization] to enable routing between hybrid nodes and external networks to allow connectivity between on-premises and AWS Cloud workloads.

 # SAlient features for BGP networking
 - BGP routing for hybrid node connectivity
 - BGP peer configuration for on-premise networks
 - Different ASN configure for each [ dev / staging / prod ] environment
 - automated advertizement for pod CIDRs

 # Manual Deployment
 ``` bash
 helm repo add cilium https://helm.cilium.io/
 helm repo update

 # install BGP CRD
 kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.17.4/pkg/k8s/apis/cilium.io/client/crds/v2alpha1/ciliumloadbalancerippools.yaml

 kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.17.4/pkg/k8s/apis/cilium.io/client/crds/v2alpha1/ciliumbgpclusterconfigs.yaml

 kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.17.4/pkg/k8s/apis/cilium.io/client/crds/v2alpha1/ciliumbgppeerconfigs.yaml

 kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.17.4/pkg/k8s/apis/cilium.io/client/crds/v2alpha1/ciliumbgpadvertisements.yaml

 # Install with helm

 cd cilium
 helm dependency build base/
 # for dev
 helm install cilium ./base -n kube-system -f config/dev/us-east-1/values.yaml
 # for test
 helm install cilium ./base -n kube-system -f config/test/us-east-1/values.yaml
 # for prod
 helm install cilium ./base -n kube-system -f config/prod/us-east-1/values.yaml

 kubectl get pods -n kube-system -l k8s-app=cilium
 kubectl get pods ciliumbgpclusterconfig,ciliumbgppeerconfig


 # Troubleshooting tips
 # 1. Service availability check
 kubectl expose pod <pod-name> --port=80 --target-port=8080
 kubectl port-forward service/<service-name> 8080:80
 
 # 2. test pod connectivity
 kubectl run test-pod --image=ngnix --overrides='{"spec":{"nodeSelector":{"eks.amazonaws.com/compute-type":"hybrid"}}}'

 # 3. check cilium status
 kubectl exec -n kube-system <cilium-pod> -- cilium-dbg status
 kubectl exec -n kube-system <cilium-pod> -- cilium-dbg bgp peers

 # 4. node affinity check

 kubectl get nodes --show-labels | grep "eks.amazon.com/compute-type=hybrid"
 kubectl get pods -n kube-system -l k8s-app=cilium -o wide

 # 5. check available routes for advertisement
 kubectl exec -n kube-system <cilim-pod> --cilium-dbg gp routes available ipv4 unicast
 # [display the IPv4 unicast routes currently available in the BGP Control Plane's Routing Information Bases (RIBs) # on a Cilium-enabled node.]
 # check pod ip allocation
 kubectl get pods -o wide | grep <pod-name>
 kubectl get ciliumnodes.cilim.io # get CIDR allocation

 # 6. hubble
 kubectl port-forward -n kube-system svc/hubble-ui 12000:80
 kubectl exec -n kube-system <cilium-pod> -- hubble observe
 ```