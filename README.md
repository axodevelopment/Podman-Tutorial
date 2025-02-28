# Podman - Tutorial

Very high level overview of and quick tour of podman.

## Additional commands

Great way to debug which ports are are being used in the pod.
nsenter -t takes in the pid of the inspect command

ss -pant is the command used to check sockets

If you don't nsenter you'll be in the logged in ns, instead of the pods ns

```bash
podman inspect container -f '{{.State.Pid}}'

sudo nsenter -t blah -n ss -pant
```

## Basic Commands

High level commands

```bash
podman

podman images

podman container list --all

podman container rm --all

podman network ls
```

---
## More exploratory commands

How to run some images in podman:
```bash
podman run --name c-basic-webapi -p 7000:6720 --network c-isolated --user 1001 docker.io/axodevelopment/rht-basic-webapi:latest

podman exec -it c-basic-webapi whoami
```

For mounts that mount into a pod or for rootless container isolation these commands can further help you diagnose or ensure isolate.

Get the Pid from the container your interested in.

```bash
podman inspect container -f '{{.State.Pid}}'
```

Find the mount namespaces associated with the ppid and show the parent ppid

```bash
lsns -t mnt -n -o PPID
```

Show which user owns the the PPID

```bash
stat -c '%u' /proc/$PPID
```

Compare the uid to is mapped with subuid in rootless case

```bash
cat /etc/subuid
```

unshare puts you into the same user namespacethat podman uses for rootless...

```bash
podman unshare cat /proc/self/uid_map
```

Explore the capabilities.

```bash
capsh --print

podman exec <container> capsh --print
```

---

## Some networking commands

How do we create and inspect a network

```bash
podman container -rm -a

podman container list --all

podman network ls

podman network rm c-isolated

podman network create c-isolated

podman network inspect c-isolated
```

How you can attack a network to a pod

```bash
podman run --name c-basic-webapi -p 7000:6720 --network c-isolated --user 1001 docker.io/axodevelopment/rht-basic-webapi:latest

podman run --name c-basic-dataapi --network c-isolated --user 1001 docker.io/axodevelopment/rht-basic-dataapi:latest
```

\*\*\* shut down web api

explore journalctl

```bash
journalctl -f

grep Storage /etc/systemd/journald.conf

#Storage=auto is defualt
#Storage=persistent for persistent storage
SystemMaxUse=500M

systemctl restart systemd-journald
```

## Podman can also do pods

How do these commands translate to pods

```bash
podman pod create

podman pod list

podman pod rm ???

podman network create p-isolated

podman pod create --name thepod -p 7007:6720 --network p-isolated

podman pod inspect thepod

podman run -dt --pod thepod docker.io/axodevelopment/rht-basic-webapi:latest

podman ps -a --pod

podman run -dt --pod thepod docker.io/axodevelopment/rht-basic-dataapi:latest

podman ps -a --pod

podman pod rm thepod --force
```

--- MOVING TO Service

```bash
loginctl user-status container

ls /var/lib/systemd/linger

loginctl disable-linger container

ls /var/lib/systemd/linger

loginctl enable-linger container

loginctl user-status container


---

mkdir -p .config/systemd/user

---

cd .config/systemd/user

rm c...

podman container ls -a

podman container stop c-basic-dataapi

podman generate

podman generate kube c-basic-dataapi

podman generate systemd --name c-basic-dataapi --files

cat container-c-basic-dataapi.service

--- show webapi again with no innerdata

systemctl --user daemon-reload

systemctl --user enable --now container-c-basic-dataapi.service

systemctl --user status container-c-basic-dataapi.service

--- show webapi again with with innerdata

--- cleanup

service=YOUR_SERVICE_NAME;
systemctl --user stop $service
systemctl --user disable $service

systemctl --user daemon-reload
systemctl --user reset-failed

rm /etc/systemd/system/$service
```

--- if there is time show dive

I like dive here is a tool to inspect the layers of images

```bash
dive podman://
```
