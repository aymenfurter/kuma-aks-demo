# Kuma Multizone Demo

## Setup KumaCTL
First, setup kuma locally. The easiest way is to use the installer (through ``curl -L https://kuma.io/installer.sh | VERSION=2.0.2 sh -``)

## Prerequesites
3 AKS Clusters are required. "aks1" will host the global control plane. "aks2" and "aks3" are both seperate zones hosting workloads.

## Setting up Kuma
For the Global Control plane you can use: ``kumactl install control-plane --mode=global | kubectl apply -f -``

Next, determind the Kuma LB IP Address of your Global Kuma Control Plan:
``kubectl get service --namespace kuma-system``

Then, for aks2 and aks3, run (update the IP accordingly): ``kumactl install control-plane \
                                --mode=zone \
                                --zone=aksx \
                                --ingress-enabled \
                                --egress-enabled \
                                --kds-global-address grpcs://192.168.0.1:5685 | kubectl apply -f ``

After deployment, add the following Annotation to the service definition
	service.beta.kubernetes.io/azure-load-balancer-internal: "true"



Now, deploy service-1 to aks1, service-2 to aks2, and service-3 to aks3 using kubectl.


Next, ensure you have mTLS enabled: ``kubectl apply -f mtls.yml``

And allow all communication using ``kubectl apply -f allowall.yml``

Next, you can test the cross-zone communication, using for instance:
Running on "aks3":
``$ curl service-2_service2_svc_80.mesh:80``

Running on "aks2":
``$ curl service-3_service3_svc_80.mesh:80``


## Restricting the traffic
To restrict the traffic (e.g. only allow traffic from service2 to service3), you can try out the following trafficpermission (make sure to delete "allowall" beforehand): ``kubectl apply -f single-direction.yml``

Once implemented, requests will only be successful from service-2 pod to service-3 service.

