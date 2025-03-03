# docker
ssh jumphost
id

uid=267988(helsens) gid=11349(UPOATES-StaffU)

docker build . --tag registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 --tag registry.rcp.epfl.ch/upoates-helsens/cellpose-env:latest \
    --build-arg LDAP_GROUPNAME=UPOATES-StaffU \
    --build-arg LDAP_GID=11349 \
    --build-arg LDAP_USERNAME=helsens \
    --build-arg LDAP_UID=267988

try the container
docker run -it registry.rcp.epfl.ch/upoates-helsens/cellpose-env:v0.1 bash


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