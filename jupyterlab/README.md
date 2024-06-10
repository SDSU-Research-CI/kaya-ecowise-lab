# JupyterLab Job
The following guide will walk users through scheduling and connecting to a JupyterLab instance on the TIDE cluster.

This guide assumes that the user has completed the [Getting Access](https://sdsu-research-ci.github.io/softwarefactory/gettingaccess) directions and installed [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) on their local machine.

Please make sure to have a local copy of this repository. You can get this repo locally by cloning with git:
```
git clone https://github.com/SDSU-Research-CI/kaya-ecowise-lab.git
```

Before running the examples below, please open this repository in your terminal and navigate to the jupyterlab directory:

```
cd jupyterlab
```

## Deploy

### Step 1: Set Context

If you wish to not pass the namespace name for each command, run the following to set the namespace context:

```
kubectl config set-context nautilus --namespace=sdsu-kaya-ecowise-lab
```

Note: All the examples below still include the `-n NAMESPACE` for convenience. 

### Step 2: Get Individual PVC Name
Run the following to see your volume/PVC:

```
kubectl get pvc -n sdsu-kaya-ecowise-lab
```

You should see one in the form of `jupyter-volume-[your-sdsuid-prefix]`, for example:

```
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           AGE
jupyter-volume-ssurakasi   Bound    pvc-861622d4-44ab-4523-b3e6-3eb3dd2548f0   250Gi      RWO            rook-ceph-block-tide   38s
```

You will want to copy the value from the NAME column to be used in the next step.

### Step 3: Customize and Schedule the JupyterLab Job

#### Customize the Job
IMPORTANT: Create a copy of the jupyter-job-L40.yaml file and modify the "jupyterpod-username" name in the file (line 4) replacing "username" with your SDSUid prefix and "jupyter-volume-username" (line 59) replacing "jupyter-volume-username" with what you copied from the previous step. If you don't do this, you may conflict with someone else using the default name.

Optional: You can increase the CPU, RAM and GPU resources on lines 38-48.
Before increasing the defaults, please monitor your resource utilization using the dashboards below:
- [CPU and RAM](https://grafana.nrp-nautilus.io/d/85a562078cdf77779eaa1add43ccec1e/kubernetes-compute-resources-namespace-pods?orgId=1&refresh=10s&var-datasource=default&var-cluster=&var-namespace=sdsu-kaya-ecowise-lab)
- [GPU](https://grafana.nrp-nautilus.io/d/dRG9q0Ymz/k8s-compute-resources-namespace-gpus?orgId=1&refresh=30s&var-namespace=sdsu-kaya-ecowise-lab)
If you are above 90% utilization, then you might consider increasing the default resources.

Requests are the minimum amount of resources your job needs in order to run.
Limits are the maximum amount of resources that your job may consume.
Limits for CPU and RAM must be less than or equal to 1.2x of the request. 
- I.E: If you request 10Gi RAM, your max limit should be 12Gi RAM.

#### Schedule the Job
Schedule the job with the following command:

```
kubectl create -f jupyter-job-L40.yaml -n sdsu-kaya-ecowise-lab
```

## Accessing JupyterLab

Get the pod for your JupyterLab instance and ensure it is running:

```
kubectl get pods -n sdsu-kaya-ecowise-lab --watch
```
- Your pod is ready when the READY columns shows '1/1' and STATUS column shows 'Running'
- You can stop the watch command with `ctrl + c`

**Note: Replace "username" with your username in the example below.**

Look for the pod with your username in it. Once running, collect your pod's logs:

```
kubectl logs jupyterpod-username -n sdsu-kaya-ecowise-lab
```

From the output, copy the URL to the clipboard that starts with http://127.0.0.1./. Paste this into a new tab in your browser, but wait to hit enter or otherwise navigate to it.

Next, run the following command to set up port forwarding between your computer and the container running Jupyter.

```
kubectl port-forward jupyterpod-username -n sdsu-kaya-ecowise-lab 8888:8888
```

*Note*: If you get an error message indicating port 8888 is already taken, simply change the port number on the left side of the colon in the above command i.e. `8889:8888`.
Then when you paste the URL from the output, make sure to update the port from 8888 to 8889, or whatever number you chose.
This issue can arise if you have a local instance of Jupyter Lab already running on port 8888.

Now switch back to your browser and hit enter, or otherwise navigate to the URL you entered previously.

## Shutdown / Cleanup

When done, delete your JupyterLab Job:

```
kubectl delete -f jupyter-job-L40.yaml -n sdsu-kaya-ecowise-lab
```

Also, make sure to end the port-forwarding process by pressing `ctrl + c` in the terminal window running that command.

`Note: Your volume will persist so you can start the pod again and have acecss to your data.`
