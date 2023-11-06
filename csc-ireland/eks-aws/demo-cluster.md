CLUSTER_NAME=c4-eks-aws-3
mkdir -p $CLUSTER_NAME
cat <<EOF > $CLUSTER_NAME/$CLUSTER_NAME.yaml
# Please change the name, region, vpc, subnet, instance type and other specs
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: ${CLUSTER_NAME}
  region: eu-west-1
vpc:
  publicAccessCIDRs: ["0.0.0.0/0"]
  id: "vpc-00bc8b021dafb7a92"  # (optional, must match VPC ID used for each subnet below)
  cidr: "172.26.0.0/22"       # (optional, must match CIDR used by the given VPC)
  clusterEndpoints:
    publicAccess: true
    privateAccess: true
  subnets:
    # must provide 'private' and/or 'public' subnets by availibility zone as shown
    private:
      subnet1:
        id: "subnet-05b6a79635031102c"
        cidr: "172.26.2.0/26" # (optional, must match CIDR used by the given subnet)

      subnet2:
        id: "subnet-029718643080a4b86"
        cidr: "172.26.2.64/26"  # (optional, must match CIDR used by the given subnet)

      subnet3:
        id: "subnet-01c68677ddab29f0c"
        cidr: "172.26.2.128/26"  # (optional, must match CIDR used by the given subnet)
nodeGroups:
  - name: md-0
    instanceType: t2.medium
    amiFamily: Ubuntu2004
    ami: ami-062ebd5f10a9d1a90
    privateNetworking: true
    desiredCapacity: 1
    securityGroups:
      attachIDs: ["sg-080b7c006220a6283"]
    volumeSize: 50
    ssh:
      publicKeyName: "nova-keypair-ssh"
    overrideBootstrapCommand: |
      #!/bin/bash
      /etc/eks/bootstrap.sh ${CLUSTER_NAME}
EOF
eksctl create cluster -f $HOME/$CLUSTER_NAME/$CLUSTER_NAME.yaml --kubeconfig=$HOME/$CLUSTER_NAME/$CLUSTER_NAME-eks-cluster.kubeconfig
KUBECONFIG=$HOME/$CLUSTER_NAME/$CLUSTER_NAME-eks-cluster.kubeconfig
kubectl get nodes