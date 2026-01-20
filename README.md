# Kubernetes CronJob for Jenkins Performance Data Collection

## Setup Instructions



### Step 1: Review and Customize Configuration

Customize these settings in `cronjob-performance.yaml`. The example uses a controller with pod label "tenant=controller-2" in namespace cloudbees-core. Change this label to match your controller:

```bash
NAMESPACE="cloudbees-core"              
LABEL_SELECTOR="tenant=controller-2"    
TARGET_CONTAINER="jenkins"              
```

The cronjob runs latest kubectl, check for compatibility with your cluster version.
 

### Step 2: Create service account/role to run the cronjob

```bash
kubectl apply -f sa.yaml
```

### Step 3: Apply ConfigMap - contains the script

```bash
kubectl apply -f script-performance.yaml
```

### Step 4: Apply CronJob - runs the script

```bash
kubectl apply -f cronjob-performance.yaml
```

### Check CronJob Status

```bash
kubectl -n cloudbees-core get cronjob exec-performance-script -o wide
```

### Manually Trigger a Test Job

Create a one-time job from the CronJob template to test immediately without waiting 5 minutes:

```bash
kubectl -n cloudbees-core create job --from=cronjob/exec-performance-script test-performance-job
```

### Check Output in Jenkins Pod

The performance data archive is written to `/tmp` inside the Jenkins container:

```bash
# List archives in the Jenkins pod
POD_NAME=$(kubectl -n cloudbees-core get pods -l tenant=controller-2 -o jsonpath='{.items[0].metadata.name}')
kubectl -n cloudbees-core exec $POD_NAME -c jenkins -- ls -lh /tmp/performanceData*.tar.gz
```

### Copy the archive from the container to your local machine

```bash
POD_NAME=$(kubectl -n cloudbees-core get pods -l tenant=controller-2 -o jsonpath='{.items[0].metadata.name}')
kubectl -n cloudbees-core cp "cloudbees-core/$POD_NAME:/tmp/performanceData.*.tar.gz" ./
```

### Change Execution Schedule

Edit `cronjob-performance.yaml`, modify the `spec.schedule` field:

```yaml
spec:
  schedule: "*/10 * * * *"  # Every 10 minutes
```
