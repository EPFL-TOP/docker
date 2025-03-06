# docker

## Copy data to RCP cluster

To copy data to the RCP cluster
```bash
ssh <USERNAME>@haas001.rcp.epfl.ch
```

id

uid=267988(helsens) gid=11349(UPOATES-StaffU)

docker build . --tag registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 --tag registry.rcp.epfl.ch/upoates-helsens/cellpose-env:latest \
    --build-arg LDAP_GROUPNAME=UPOATES-StaffU \
    --build-arg LDAP_GID=11349 \
    --build-arg LDAP_USERNAME=helsens \
    --build-arg LDAP_UID=267988


docker build . --tag registry.rcp.epfl.ch/upoates-helsens/plantseg-env:v0.1 \
    --build-arg LDAP_GROUPNAME=UPOATES-StaffU \
    --build-arg LDAP_GID=11349 \
    --build-arg LDAP_USERNAME=helsens \
    --build-arg LDAP_UID=267988

try the container
docker run -it registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 bash
docker run -it registry.rcp.epfl.ch/upoates-helsens/plantseg-env:v0.1 bash


to register to harbor
docker login registry.rcp.epfl.ch

docker push registry.rcp.epfl.ch/upoates-helsens/cellpose-env:latest
docker push registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1



Login to runAI
runai login

list projects
runai list project


PVC -> upoates-scratch
kubectl get pvc -n  runai-upoates-helsens
NAME              STATUS   VOLUME                    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
home              Bound    upoates-home-helsens      100Ti      RWX                           9d
upoates-scratch   Bound    upoates-scratch-helsens   100Ti      RWX                           9d


kubectl get secrets for private images
NAME                 TYPE                             DATA   AGE
my-registry-secret   kubernetes.io/dockerconfigjson   1      25h

runai submit \
  --name cellpose-test \
  --image registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 \
  --gpu 0.5 \
  --environment MY_ENV_VAR="A test ENV variable" \
  --existing-pvc claimname=upoates-scratch,path=/scratch \
  --existing-pvc claimname=home,path=/home/helsens \
  --command \
  -- /bin/bash -ic "sleep 6000"


connect to the job container when running
runai bash my-demo-job


 runai submit \                                                               
  --name cellpose-test-6 \
  --image registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 \
  --gpu 1. \
  --existing-pvc claimname=upoates-scratch,path=/scratch \
  --command \
  -- /usr/bin/python3 /home/helsens/3d_segmentation/3d_cellpose.py /scratch/data/feyza/pos1_Channel1_cropped /scratch/data/feyza/cellpose_training/2d/models/nuclei_h2b


  runai submit \                                                               
  --name plantseg-test-1 \
  --image registry.rcp.epfl.ch/upoates-helsens/plantseg-env:v0.1 \
  --gpu 1. \
  --existing-pvc claimname=upoates-scratch,path=/scratch \
  --command \
  -- conda init \
  -- bash

