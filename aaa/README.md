# Introduction

Nitty-gritty of API request processing.

## Authentication, authorization and admission control

Every kubernetes request to API server goes through the following three stages:

- authentication: check if the identity of the user is valid
- authorization: check if the user has proper permission to access the resource
- admission control: module to validate or modify the request

## Authentication



## Authorization

## Admission control

## Excercise

Start minikube and verify config:

    minikube start
    kubectl config view

Create namespace `fermat`:

    kubectl create namespace fermat;

Create public/private key for student under `rbac` directory:

    mkdir rbac
    cd rbac
    openssl genrsa -out student.key 2048
    openssl req -new -key student.key -out student.csr -subj "/CN=student/O=learner"

Create a YAML to sign the csr generated in previous step by kubernetes:

    cat <<EOF > signing-request.yaml
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: student-csr
    spec:
      groups:
      - system:authenticated
      request: $(cat student.csr | base64 | tr -d '\n')
      signerName: kubernetes.io/kube-apiserver-client
      usages:
      - digital signature
      - key encipherment
      - client auth

    EOF

Create the certificate signing request:

    kubectl create -f signing-request.yaml

Review the csr object:

    kubectl get csr

    NAME          AGE    SIGNERNAME                            REQUESTOR       CONDITION
    student-csr   7m3s   kubernetes.io/kube-apiserver-client   minikube-user   Pending

Approve the csr and review the object again:

    kubectl certificate approve student-csr
    kubectl get csr

    NAME          AGE     SIGNERNAME                            REQUESTOR       CONDITION
    student-csr   9m46s   kubernetes.io/kube-apiserver-client   minikube-user   Approved,Issued

Extract the approved certficate, decode it w/ base64 and save the certificate:

    kubectl get csr student-csr -o jsonpath='{.status.certificate}' | base64 -d > student.crt

Configure the `kubectl` client w/ the new credentials:

    kubectl config set-credentials student --client-certificate=student.crt --client-key=student.key

Create a new context for `student` within namespace `fermat`:

    kubectl config set-context student-context --cluster=minikube --namespace=fermat --user=student

Create a deployment in namespace `fermat`:

    kubectl -n fermat create deployment nginx --image=nginx:alpine
