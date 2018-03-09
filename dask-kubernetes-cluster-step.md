## Dask Cluster on Google Cloud

### Create a Kubernetes cluster on Google Cloud

Create a Kubernetes cluster on Google Cloud, by typing in the following command:

    gcloud container clusters create kaggle --num-nodes=3 --machine-type=n1-standard-2 --zone=us-central1-b

To test if your cluster is initialized, run:

    abanihirwe@yellowstone:~/devel/kaggle/kaggle-talkingdata-adtracking-fraud-detection$ kubectl get node
    NAME                                    STATUS    ROLES     AGE       VERSION
    gke-kaggle-default-pool-09161020-gtkz   Ready     <none>    1m        v1.8.7-gke.1
    gke-kaggle-default-pool-09161020-n963   Ready     <none>    1m        v1.8.7-gke.1
    gke-kaggle-default-pool-09161020-s2tc   Ready     <none>    1m        v1.8.7-gke.1
    abanihirwe@yellowstone:~/devel/kaggle/kaggle-talkingdata-adtracking-fraud-detection$

Give your account super-user permissions, allowing you to perform all the actions needed to set up JupyterHub.

    kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=<your-email-address>


### Setting up Helm

 Installation

    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

Initialization

After installing helm on your machine, initialize helm on your Kubernetes cluster. At the terminal, enter:

    kubectl --namespace kube-system create sa tiller
    kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
    helm init --service-account tiller

Helm Install Dask

    helm repo add dask https://dask.github.io/helm-chart
    helm repo update

Now you can launch Dask on your Kubernetes cluster using the Dask Helm chart:

    helm install dask/dask

This deploys a dask-scheduler, several dask-worker processes, and also a Jupyter server.

Verify Deployment

    abanihirwe@yellowstone:~/devel/kaggle/kaggle-talkingdata-adtracking-fraud-detection$ kubectl get pods
    NAME                                        READY     STATUS              RESTARTS   AGE
    veering-ocelot-jupyter-7945df487c-qnwz6     0/1       ContainerCreating   0          2m
    veering-ocelot-scheduler-6cbdb49dc6-zc8bt   1/1       Running             0          2m
    veering-ocelot-worker-d8cb65c5-gfcn5        1/1       Running             0          2m
    veering-ocelot-worker-d8cb65c5-r6hwv        1/1       Running             0          2m
    veering-ocelot-worker-d8cb65c5-rpd6q        0/1       ContainerCreating   0          2m
    abanihirwe@yellowstone:~/devel/kaggle/kaggle-talkingdata-adtracking-fraud-detection$ kubectl get pods
    NAME                                        READY     STATUS    RESTARTS   AGE
    veering-ocelot-jupyter-7945df487c-qnwz6     1/1       Running   0          2m
    veering-ocelot-scheduler-6cbdb49dc6-zc8bt   1/1       Running   0          2m
    veering-ocelot-worker-d8cb65c5-gfcn5        1/1       Running   0          2m
    veering-ocelot-worker-d8cb65c5-r6hwv        1/1       Running   0          2m
    veering-ocelot-worker-d8cb65c5-rpd6q        1/1       Running   0          2m
    abanihirwe@yellowstone:~/devel/kaggle/kaggle-talkingdata-adtracking-fraud-detection$ kubectl get services
    NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)AGE
    kubernetes                 ClusterIP      10.43.240.1    <none>           443/TCP13m
    veering-ocelot-jupyter     LoadBalancer   10.43.254.47   35.225.254.167   80:31746/TCP2m
    veering-ocelot-scheduler   LoadBalancer   10.43.241.39   35.194.26.249    8786:31932/TCP,80:32597/TCP2m

Connect to Dask and Jupyter

    abanihirwe@yellowstone:~/devel/kaggle/kaggle-talkingdata-adtracking-fraud-detection$ kubectl get services
    NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)AGE
    kubernetes                 ClusterIP      10.43.240.1    <none>           443/TCP19m
    veering-ocelot-jupyter     LoadBalancer   10.43.254.47   35.225.254.167   80:31746/TCP8m
    veering-ocelot-scheduler   LoadBalancer   10.43.241.39   35.194.26.249    8786:31932/TCP,80:32597/TCP8m

We can navigate to these from any web browser. One is the Dask diagnostic dashboard. The other is the Jupyter server. You can log into the Jupyter notebook server with the password, dask

### Configure Environment

    helm upgrade veering-ocelot dask/dask -f config.yaml
