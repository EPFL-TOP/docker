# docker

## Copy data to RCP cluster

To copy data to the RCP cluster, need to have first copied the data on the NAS-RCP. Either the data is already in the raw data catalog and thus store on the NAS, or it needs to be copied. A folder exist for that purpose, it is located inside `common/ForRCP`. In this folder every user creates is own folder, for example `clement` and creates two sub-folders `input` and `output`. Resulting in something like `common/ForRCP/clement/input` and `common/ForRCP/clement/output`. In the input folder we will add all the data that we need to be copied to the cluster for processing, in the output folder, all the data produced during processing will be copied back (for example segmentation masks, model, text file). The cluster has a high performance scratch space of 2.5TB, but nothing is backup, and can be deleted when space is needed (for example when many TBs are sitting there for long time w/o accessing them). The only machine that has an access to both the lab NAS and the RCP scratch is the `JumpHost`. To connect to it, simply replace `<USERNAME>` with your gaspar and type in a terminal:

```bash
ssh <USERNAME>@haas001.rcp.epfl.ch
```

On this machine, our lab NAS is mounted as `/mnt/upoates/collaborative` and the scratch (accessible to the GPU cluster) as `/mnt/upoates/scratch`. If you want to list the files for RCP, do something like:

```bash
ls /mnt/upoates/collaborative/ForRCP
```

What we want at this point is to replicate all the files that we have inside `/mnt/upoates/collaborative/ForRCP` on the cluster's scratch `/mnt/upoates/scratch`. For that let's create the same structure. 


id

uid=267988(helsens) gid=11349(UPOATES-StaffU)




docker buildx build --platform linux/amd64 --tag registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.2  --build-arg LDAP_GROUPNAME=UPOATES-StaffU --build-arg LDAP_GID=11349 --build-arg LDAP_USERNAME=helsens  --build-arg LDAP_UID=267988 .

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

docker run -it --mount type=bind,source=/Volumes/upoates,target=/mydata registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 bash

to register to harbor
docker login registry.rcp.epfl.ch

docker push registry.rcp.epfl.ch/upoates-helsens/cellpose-env:latest
docker push registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1

get kube config the file
curl https://wiki.rcp.epfl.ch/public/files/kube-config.yaml -o ~/.kube/config && chmod 600 ~/.kube/config

set the cluster
runai config cluster rcp-caas-prod

Login to runAI
runai login

list projects
runai list project

runai config project upoates-helsens

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

use this to login with different credentials
  --run-as-uid UID \
  --run-as-gid GID \