# LAB TITLE: AK8S-05 Upgrading Kubernetes Engine Clusters

## OBJECTIVES: 
	-Use the GCP Console to upgrade your GKE cluster.


To achieve the above objective we need to perform 2 tasks:
	-Task 1. Deploy a GKE cluster.
	-Task 2: Upgrade your GKE cluster.

To deploy a GKE cluster I will run the cmd- gcloud container clusters create cluster-1 --zone us-central1-c --cluster-version 1.15.12-gke.2 --num-nodes "3" --machine-type "n1-standard-1" --enable-ip-alias --default-max-pods-per-node "110"

To upgrade the GKR cluster I will run the cmd- gcloud container clusters upgrade cluster-1 --node-pool=default-pool --cluster-version=latest --zone=us-central1-c
