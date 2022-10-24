# Connect to private regional cluster via bastion host

I'm using this tutorial here:
*  https://cloud.google.com/kubernetes-engine/docs/tutorials/private-cluster-bastion

Whenever I'm restarting working on it, I'm setting these variables:


```bash
export CLUSTER_NAME=
export REGION=us-central1
export COMPUTE_ZONE=us-central1-a
export SUBNET_NAME=my-subnet-0
export INSTANCE_NAME=
export PROJECT_ID=
```

Start by creating a cluster:

```bash
gcloud container clusters create $CLUSTER_NAME \
    --create-subnetwork name=$SUBNET_NAME \
    --region=$REGION \
    --enable-master-authorized-networks \
    --enable-ip-alias \
    --enable-private-nodes \
    --enable-private-endpoint \
    --master-ipv4-cidr 172.16.0.32/28
```

This cluster created is private and regional. Can I connect to its control plane?

```bash
kubectl get nodes # times out
```

Now I will set up a bastion host. Create a Compute Engine VM:

```bash
gcloud compute instances create $INSTANCE_NAME \
    --zone=$COMPUTE_ZONE \
    --machine-type=e2-micro \
    --network-interface=network-tier=PREMIUM,subnet=$SUBNET_NAME
```

Start session into the bastion host to install `tinyproxy`:

```bash
gcloud compute ssh $INSTANCE_NAME --tunnel-through-iap --project=$PROJECT_ID --zone=$COMPUTE_ZONE
```

If you get this issue connecting:
>`ERROR: (gcloud.compute.ssh) [/usr/local/bin/ssh] exited with return code [255].


You can connect from Pantheon. Or you can just retry. Once connected start installing `Tinyproxy`:

```bash
sudo apt install tinyproxy # this installation took at least several minutes on the master VM
# then open tinyproxy config file:
sudo vi /etc/tinyproxy/tinyproxy.conf
# add to the Allow section
Allow localhost
# restart tinyproxy
sudo service tinyproxy restart
```
Now get credentials to the cluster:

```bash
gcloud container clusters get-credentials $CLUSTER_NAME \
    --region=$REGION \
    --project=$PROJECT_ID
```

Try tunneling to the bastion host using IAP:

```bash
gcloud beta compute ssh $INSTANCE_NAME \
    --tunnel-through-iap \
    --project=$PROJECT_ID \
    --zone=$COMPUTE_ZONE \
    -- -4 -L8888:localhost:8888 -N -q -f
```

In my case tuneling through IAP did not work, but the same command without `--tunnel-through-iap` worked:

```bash
gcloud beta compute ssh $INSTANCE_NAME \
    --project=$PROJECT_ID \
    --zone=$COMPUTE_ZONE \
    -- -4 -L8888:localhost:8888 -N -q -f
```
After than set `HTTPS_PROXY` on the system level:

```bash
export HTTPS_PROXY=localhost:8888
```

Now I'm able to execute kubectl commands against my private cluster:

```bash
kubectl get nodes
```
