# Manage-multiple-cluster-KOPS-GKE
Managing multi cloud cluster using KOPS + GKE 

##  Pre-requisites 

1.  One Ubuntu Virtual Machine (AWS/GCP/AZURE/Virtualbox etc) with AWS CLI + GCLOUD installed. 
    (Centos or any LX flavour with work. Demo uses Ubuntu)

2.  aws configure executed successfully 

3.  gcloud init configured successfully 

4.  One globally accessible DNS - kubernetesfederatedcluster.com will be used in this demo

##  A.  Setup KOPS on the machine 

    1.  wget https://github.com/kubernetes/kops/releases/download/1.10.0/kops-linux-amd64
    
    2.  chmod +x kops-linux-amd64

    3.  mv kops-linux-amd64 /usr/local/bin/kops

    4. Edit .bashrc to add /usr/local/bin/ in $PATH. Reload .bashrc using - source .bashrc
    
##  B.  Setup kubectl on the machine

    1.  sudo apt-get update && sudo apt-get install -y apt-transport-https

    2.  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

    3.  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

    4.  sudo apt-get update

    5.  sudo apt-get install -y kubectl

##  C.  Create cluster in AWS using KOPS

    1.  ssh-keygen to generate ssh keys if not exists. provide accurate permissions (700 to .ssh and 600 to .ssh/*)
    
    2.  cat id_rsa.pub > authorized_keys; chmod 600 authorized_keys;
    
    3.  Create a route53 domain on AWS -
    
        a.  Multiple clusters can be created by using subdomain as a part of hosted zones
        
        b.  Create a route53 hosted zone using CLI - aws route53 create-hosted-zone --name kubernetesfederatedcluster.com --caller-reference 1
        
                                                              OR
                                                              
        c.  Create a route53 hosted zone using AWS GUI
        
        d.  Edit the nameserver settings from your DNS provider and set the nameservers provided by AWS 
        
        e.  Verify the nameserver settings by running : dig NS kubernetesfederatedcluster.com
        
    4.  Create a S3 bucket to store your state of the cluster - 
    
        aws s3 mb s3://clusters.kubernetesfederatedcluster.com 
        
    5.  export KOPS_STATE_STORE=s3://clusters.kubernetesfederatedcluster.com  -- This is done to set the S3 storage as default 
    
    6.  Build cluster configuration - kops create cluster --zones=us-east-1c kubernetesfederatedcluster.com
    
    7.  List your Cluster using --  kops get cluster
    
    8.  Create the cluster using -- kops update cluster kubernetesfederatedcluster.com
    
    9.  Use kubectl to get list of nodes / pods for verification 
    
    10. Apply the CNI of your choice
    
##  D.  Save config of KOPS cluster

    1.  mv .kube .kube_AWS
    
##  E.  Create GKE cluster

    1.  Execute the below command to create a GKE standard 3 node cluster 
    
       gcloud beta container --project "azurestack" clusters create "standard-cluster-1" --zone "us-central1-a" --username "admin" --cluster-version "1.11.7-gke.4" --machine-type "custom-1-2048" --image-type "COS" --disk-type "pd-standard" --disk-size "20" --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --enable-cloud-logging --enable-cloud-monitoring --no-enable-ip-alias --network "projects/azurestack/global/networks/default" --subnetwork "projects/azurestack/regions/us-central1/subnetworks/default" --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair
       
    2.  Verify the cluster using kubectl command 
    
##  F.  Save config of GKE cluster

    1.  cd .kube 
    
    2.  mv config config_GCP
    
##  G.  Merge both the configs
    
    1.  cp ../.kube_AWS/config config_AWS 
    
    2.  export KUBECONFIG=":/root/.kube/config_GCP:/root/.kube/config_AWS"
    
    3.  kubectl config view -- Verify you are getting information of both the cluster 
    
    4.  kubectl config view --flatten > /root/.kube/config  -- Merge both the config to a single config 
    
    5.  kubectl config get-contexts -- Verify if you can see both the contexts 
    
    6.  kubectl config use-context CONTEXT_NAME -- where CONTEXT_NAME is the name of the cluster you want to use






    
