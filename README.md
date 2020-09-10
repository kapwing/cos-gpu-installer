Create the instance to convert to a machine image
``` shell
gcloud compute instances create gpu-instance-fresh-template \
    --zone=us-west1-b \
    --image-project cos-cloud \
    --image-family cos-85-lts \
    --subnet=projects/kapwing-181323/regions/us-west1/subnetworks/worker-subnet-us-west1 \
    --accelerator "type=nvidia-tesla-t4,count=4" \
    --machine-type custom-24-92160 \
    --boot-disk-size=50GB \
    --boot-disk-type=pd-ssd \
    --min-cpu-platform="Intel Skylake" \
    --maintenance-policy=TERMINATE \
    --network-tier=PREMIUM \
    --tags=docker-server \
    --shielded-vtpm \
    --shielded-integrity-monitoring \
    --metadata-from-file "cos-gpu-installer-env=scripts/gpu-installer-env,user-data=install-test-gpu.cfg,run-installer-script=scripts/run_installer.sh,run-cuda-test-script=scripts/run_cuda_test.sh"
```
Wait about 10 minutes after running because the nvidia drivers need to install

Stop the instance

```
gcloud compute instances stop gpu-instance-fresh-template --zone=us-west1-b
```

Create the Image (not to be confused with a Machine Image)
``` shell
gcloud compute images create gpu-image-fresh \
    --source-disk=gpu-instance-fresh-template \
    --source-disk-zone us-west1-b
```
(Instance can be deleted now.)

# Original readme below
# GPU Driver Installer containers for Container-Optimized OS from Google

Note: This is not an official Google product.

This repository contains scripts to build Docker containers that can be used to
download, compile and install GPU drivers on
[Container-Optimized OS](https://cloud.google.com/container-optimized-os/) images.

## How to use

Example command:
``` shell
gcloud compute instances create $USER-cos-gpu-test \
    --image-family cos-stable \
    --image-project cos-cloud \
    --accelerator=type=nvidia-tesla-k80 \
    --boot-disk-size=25GB \
    --maintenance-policy=TERMINATE \
    --metadata-from-file "cos-gpu-installer-env=scripts/gpu-installer-env,user-data=install-test-gpu.cfg,run-installer-script=scripts/run_installer.sh,run-cuda-test-script=scripts/run_cuda_test.sh"
```

The command above creates a GCE instance based on cos-stable image. Then it
installs GPU driver on the instance by running a container 'cos-gpu-installer'
which is implemented in this repository.

The GPU driver version and container image version are specified in
scripts/gpu-installer-env. You can edit the file if you want to install
GPU driver version or use container image other than the default.
