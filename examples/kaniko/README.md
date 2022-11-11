# README

[Kaniko](https://github.com/GoogleContainerTools/kaniko) is an OSS project that allows you to build container images in Kubernetes.  This is often used with CI/CD or GitOps pipelines.

This example makes use of [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) to get authorization to push to [GCR](https://cloud.google.com/container-registry), but you can alternatively [use GCP credentials passed as a k8s secret](https://github.com/GoogleContainerTools/kaniko/blob/main/README.md#kubernetes-secret).

**IMPORTANT**: your GKE cluster's OAuth scopes will have to include **`https://www.googleapis.com/auth/devstorage.full_control`**

### How-To
-you will need to create a GCS bucket to store the container image context tarball (I've provided an [example](./context.tar.gz), but feel free to use your own):
```
gsutil mb gs://${GCS_BUCKET}

gsutil cp ./context.tar.gz gs://${GCS_BUCKET}
```

```
kubectl create serviceaccount wi-kaniko-user
```

- you will need an additional role added to the **basic-wi-gsa** service account that got created as part of this repo's blueprint:
```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --role roles/storage.admin \
  --member "serviceAccount:basic-wi-gsa@${PROJECT_ID}.iam.gserviceaccount.com"
```

```
gcloud iam service-accounts add-iam-policy-binding basic-wi-gsa@${PROJECT_ID}.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:${PROJECT_ID}.svc.id.goog[default/wi-kaniko-user]"
```

```
kubectl annotate serviceaccount wi-kaniko-user \
  iam.gke.io/gcp-service-account=basic-wi-gsa@${PROJECT_ID}.iam.gserviceaccount.com
```

- make a copy of the [pod YAML](./kaniko-executor-wi.yaml.sample), edit accordingly and then apply!
```
kubectl apply -f ./kaniko-executor-wi.yaml
```

- sample output (via [`stern`](https://github.com/wercker/stern)):
```
...
...
kaniko kaniko INFO[0001] No cached layer found for cmd RUN apt-get update && apt-get install -y wget unzip
kaniko kaniko INFO[0001] Unpacking rootfs as cmd RUN apt-get update && apt-get install -y wget unzip requires it.
kaniko kaniko INFO[0004] ENV VAULT_VERSION=1.11.4
kaniko kaniko INFO[0004] No files changed in this command, skipping snapshotting.
kaniko kaniko INFO[0004] ARG DEBIAN_FRONTEND=noninteractive
kaniko kaniko INFO[0004] No files changed in this command, skipping snapshotting.
kaniko kaniko INFO[0004] RUN apt-get update && apt-get install -y wget unzip
kaniko kaniko INFO[0004] Initializing snapshotter ...
kaniko kaniko INFO[0004] Taking snapshot of full filesystem...
kaniko kaniko INFO[0005] Cmd: /bin/sh
kaniko kaniko INFO[0005] Args: [-c apt-get update && apt-get install -y wget unzip]
kaniko kaniko INFO[0005] Running: [/bin/sh -c apt-get update && apt-get install -y wget unzip]
kaniko kaniko Get:1 http://deb.debian.org/debian buster InRelease [122 kB]
kaniko kaniko Get:2 http://deb.debian.org/debian-security buster/updates InRelease [34.8 kB]
kaniko kaniko Get:3 http://deb.debian.org/debian buster-updates InRelease [56.6 kB]
kaniko kaniko Get:4 http://deb.debian.org/debian buster/main amd64 Packages [7909 kB]
kaniko kaniko Get:5 http://deb.debian.org/debian-security buster/updates/main amd64 Packages [369 kB]
kaniko kaniko Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [8788 B]
kaniko kaniko Fetched 8500 kB in 2s (4465 kB/s)
kaniko kaniko Reading package lists...
kaniko kaniko Reading package lists...
kaniko kaniko Building dependency tree...
kaniko kaniko Reading state information...
kaniko kaniko The following additional packages will be installed:
kaniko kaniko   ca-certificates libpcre2-8-0 libpsl5 libssl1.1 openssl publicsuffix
kaniko kaniko Suggested packages:
kaniko kaniko   zip
kaniko kaniko The following NEW packages will be installed:
kaniko kaniko   ca-certificates libpcre2-8-0 libpsl5 libssl1.1 openssl publicsuffix unzip
kaniko kaniko   wget
kaniko kaniko 0 upgraded, 8 newly installed, 0 to remove and 0 not upgraded.
kaniko kaniko Need to get 4039 kB of archives.
kaniko kaniko After this operation, 11.1 MB of additional disk space will be used.
kaniko kaniko Get:1 http://deb.debian.org/debian buster/main amd64 libpcre2-8-0 amd64 10.32-5 [213 kB]
kaniko kaniko Get:2 http://deb.debian.org/debian buster/main amd64 libpsl5 amd64 0.20.2-2 [53.7 kB]
kaniko kaniko Get:3 http://deb.debian.org/debian buster/main amd64 wget amd64 1.20.1-1.1 [902 kB]
kaniko kaniko Get:4 http://deb.debian.org/debian buster/main amd64 libssl1.1 amd64 1.1.1n-0+deb10u3 [1551 kB]
kaniko kaniko Get:5 http://deb.debian.org/debian buster/main amd64 openssl amd64 1.1.1n-0+deb10u3 [855 kB]
...
...
```