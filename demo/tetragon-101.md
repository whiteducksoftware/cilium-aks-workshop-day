# Getting started with Tetragon

## Deny 'cat' on a specific file

First deploy the policy and a busybox to play around with:

```bash
kubectl apply -f ../src/cat-kill.yaml

kubectl run -i --tty busybox-$RANDOM --image=busybox --restart=Never -- sh
```

Then, try to read a file:

```bash
echo "test" > /tmp/myfile
cat /tmp/myfile
less /tmp/myfile
```

Review the logs:

```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout | tetra getevents -o compact
```

## Deny writing a specific file

Next deploy the policy and another a busybox:

```bash
kubectl apply -f ../src/write-file-deny.yaml

kubectl run -i --tty busybox-$RANDOM --image=busybox --restart=Never -- sh
```

Now, try to write a file:

```bash
echo "test" > /tmp/test
echo "test" > /tmp/testfile
```

Review the logs:

```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout | tetra getevents -o compact
```

## Deny writing a specific locl disk

Again deploy the policy and a busybox (this time with `--privileged=true`):

```bash
kubectl apply -f ../src/mount-deny.yaml

kubectl run -i --tty busybox-$RANDOM --image=busybox --restart=Never --privileged=true -- sh
```

Now, try to write a file:

```bash
mkdir -p /mnt/host
mount /dev/sda1 /mnt/host
mount
```

Review the logs:

```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout | tetra getevents -o compact
```
