# Heapster for Kubernetes Dashboard Graphs
So you've just setup your Kubernetes cluster and deployed the [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) to it.
You navigate to the dashboard expecting to get some info on your deployments and, more importantly, an eyeful
of those sweet, sweet graphs.  The dashboard loads up and you stare in utter disappointment at an endless wall of text.
Why the hell does _your_ Kubernetes dashboard not have graphs like every single screenshot you've seen?  You furiously
start Googling phrases like `kubernetes dashboard no graphs` or `what's up with the lack of graphs on my kubernetes dashboard`.

In short order you realize that you need to have Heapster running on your cluster in order to get those gorgeous graphs
on your dashboard.  The [official documentation](https://github.com/kubernetes/heapster) offers remarkably little help.
There's no mention whatsoever of the bare minimum requirements for getting graphs on your dashboard.  It's obvious that
you'll need Influxdb and Heapster but do you really _need_ Grafana?  Is any special setup required to get the dashboard to
"see" Heapster and start displaying those eye-meltingly beautiful graphs?  Well, friends, here have you the answers that
you so desperately seek:

* You only need Heapster and Influxdb - Grafana is entirely optional.
* As long as you've got Heapster exposed via a service called `heapster`, you're all good.

The yaml files in the [Influxdb section](https://github.com/kubernetes/heapster/tree/master/deploy/kube-config/influxdb) 
of the official [Heapster](https://github.com/kubernetes/heapster) documentation will get you close to what you need but
they simultaneously add some superfluous stuff **and** leave out a few important bits.  Instead of relying on those, checkout
the two yaml files in this project's `deploy` directory.  They're exactly what you need to add a bare minimum Heapster/Influxdb
deployment to your Kubernetes cluster.

To deploy Heapster and Influxdb, clone this project to a directory easily accessible by `kubectl`, cd into that directory
and follow these simple instructions:

### Create a PersistentVolume for Influxdb data
The Influxdb container is going to need a place to store its data.  When applied, the `deploy/influxdb.yaml` file 
will attempt to create a PersistentVolumeClaim against a PersistentVolume whose `storageClassName` is `influxdb`.
You'll need to create this [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).
Since there are many, many different ways to create persistent volumes, no template has been included in this project.

Here's an example, however, of what a PersistentVolume definition might look like if you had some theoretical NFS space
available on a hypothetical NAS server:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: influxdb-persistent-volume
  namespace: kube-system
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: influxdb
  nfs:
    path: /mnt/nfs-storage/influxdb
    server: 192.168.0.101
```

The important bits of the above example that you'll need to replicate in your persistent volume definition are as follows:

* It's in the **kube-system** namespace
* The **storageClassName** is **influxdb**

### Deploy Heapster and Influxdb
Once you've got your persistent volume in place according to the above instructions, simply run the following command to
deploy Heapster and Influxdb:

```bash
$ kubectl apply -f deploy/
```

That ought to do it.  The Heapster and Influxdb deployments will spin up one pod each which'll be named something like this:

```
NAME                                     READY     STATUS    RESTARTS   AGE
heapster-1428305041-jp8hh                1/1       Running   0          1h
monitoring-influxdb-2825711038-v0zl6     1/1       Running   0          1h
```

Check the logs of these two pods to verify that they've started up without any error messages.  Remember that these pods
are running in the **kube-system** namespace so you'll need to append `-n kube-system` to your log commands.

If both pods are running, you should be good to go.  Log into the Kubernetes Dashboard and 
[behold those dazzling graphs](https://www.youtube.com/watch?v=sIlNIVXpIns) that you're desirous of for so very long.