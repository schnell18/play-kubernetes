# Introduction

Nitty-gritty of configMap and secret.

## Reason to have configMap

Using configMap allows you to reuse container image. ConfigMap can be used as
environment variables, commands and arguments or volumes. ConfigMpa can be
create from literal, config files or directories.

## Create a configMap from literal values

    kubectl create configmap my-config \
        --from-literal=jdbcUrl=jdbc:mysql:test \
        --from-literal=jdbcUser=pilot \
        --from-literal=jdbcPassword=pilot

Display the content of configMap:

    kubectl get configmaps my-config -o yaml

## Create a configMap from config file

The example above can be rewritten to use a yaml file as follows:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: jdbc-config
    data:
      jdbcUrl: jdbc:mysql:test
      jdbcUser: pilot
      jdbcPassword: pilot

Kubernetes also support creating configMap from Java's properties. The above
example can be rewritten as:

    cat <<EOF > jdbc.properties
    jdbcUrl=jdbc:mysql:test
    jdbcUser=pilot
    jdbcPassword=pilot
    EOF

    kubectl create configmap prop-config --from-file=jdbc.properties

## Use configMap

We can use configMap as environment variables or as volumes.

### Use configMap as environment variables

We can either make an entire configMap as environment variables or just use a
specific key. Here is a snippet to expose an entire configMap as env variables:

    ...
        containers:
        - name: myapp-full-container
          image: some_image
          envFrom:
          - configMapRef:
              name: jdbc-config
    ...

Here is an example to use specific keys from two configMap:
    ...
        containers:
        - name: myapp-specific-container
          image: some_image
          env:
          - name: JDBC_USER
            valueFrom:
              configMapKeyRef:
                name: jdbc-config
                key: jdbcUser
          - name: JDBC_PASS
            valueFrom:
              configMapKeyRef:
                name: my-config
                key: jdbcPassword
    ...

Here is a complete demo of retrieving entire configMap as env variables:

    apiVersion: v1
    kind: Pod
    metadata:
      name: config-map-entire
      labels:
        app: config-map-entire
    spec:
      containers:
      - name: debian1
        image: debian:10.8
        command: ["/bin/sh", "-c", "env; sleep 600"]
        envFrom:
        - configMapRef:
            name: jdbc-config

Here is a complete demo of retrieving specific configMap key as env variables:

    apiVersion: v1
    kind: Pod
    metadata:
      name: config-map-specific
      labels:
        app: config-map-specific
    spec:
      containers:
      - name: debian2
        image: debian:10.8
        env:
        - name: JDBC_USER
          valueFrom:
            configMapKeyRef:
              name: jdbc-config
              key: jdbcUser
        - name: JDBC_PASS
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: jdbcPassword
        command: ["/bin/sh", "-c", "env; sleep 600"]

### Use configMap as volumes

We can also use configMap as volumes. For each key in the configMap, a file is
created in mount path. The content of the file is the value of the key.
Here is the snippet to use configMap as volume:

    ...
        containers:
        - name: myapp-vol-container
          image: some_image
          volumeMounts:
          - name: config-volume
            mountPath: /etc/config
        volumes:
        - name: config-volume
          configMap:
            name: jdbc-config
    ...

Here is a complete demo of mount the `jdbc-config` configMap as volume:

    apiVersion: v1
    kind: Pod
    metadata:
      name: config-map-volume
      labels:
        app: config-map-volume
    spec:
      containers:
      - name: debian3
        image: debian:10.8
        volumeMounts:
        - name: config-volume
          mountPath: /etc/appconf
        command: ["/bin/sh", "-c", "tail /etc/appconf/*; sleep 600"]
      volumes:
      - name: config-volume
        configMap:
          name: jdbc-config

## Secret

Secret help keeping sensitive information such as password secure.
By default, kubernetes store secret in plain text into etcd. For
production-grade kubernetes cluster, you should enable encryption feature so
that etcd will be source of secret exposure.

### Create literal secret

You use `kubectl create secret` command to create secret. For instance,

    kubectl create secret generic my-password --from-literal=password=mysecret

To display secret, you use either `kubectl get secret` or `kubectl describe
secret`. However, the secret content will no display to avoid accidental
exposure.

    kubectl get secret my-password

    NAME          TYPE     DATA   AGE
    my-password   Opaque   1      17s

    kubectl describe secret my-password

    Name:         my-password
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Type:  Opaque

    Data
    ====
    password:  8 bytes

### Create secret w/ yaml

You can also create secret using a yaml file.
There two representation of secret data:
- plain text: using the stringData attribute
- base64-encoded text: using the data attribute

If you have password `mysecret` to store in a kubernetes secret, you use
one of the yaml files as follows:

    apiVersion: v1
    kind: Secret
    metadata:
      name: my-password
    type: Opaque
    data:
      password: bXlzZWNyZXQK

Using `stringData`, you save the effort to base64-encode the secret text:

    apiVersion: v1
    kind: Secret
    metadata:
      name: my-password
    type: Opaque
    stringData:
      password: mysecret

### Create secret w/ file

You can also create secret using an ordinary file:

    echo myscret > password.txt
    kubectl create secret generic my-file-password --from-file=password.txt

## Use secret

Like configMap, you can use secret in pod either as data volume or as
environment variables.

Here is a complete example to demontrate secret usage:

    apiVersion: v1
    kind: Secret
    metadata:
      name: my-password-vault
    type: Opaque
    stringData:
      jdbcPassword: mysecret-db-password
      ldapPassword: mysecret-ldap

    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-as-env
      labels:
        app: secret-as-env
    spec:
      containers:
      - name: debian1
        image: debian:10.8
        command: ["/bin/sh", "-c", "env; sleep 600"]
        envFrom:
        - secretRef:
            name: my-password-vault

    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-as-env2
      labels:
        app: secret-as-env2
    spec:
      containers:
      - name: debian2
        image: debian:10.8
        env:
        - name: JDBC_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-password-vault
              key: jdbcPassword
        - name: LDAP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-password-vault
              key: ldapPassword
        command: ["/bin/sh", "-c", "env; sleep 600"]

    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-as-vol
      labels:
        app: secret-as-vol
    spec:
      containers:
      - name: debian3
        image: debian:10.8
        command: ["/bin/sh", "-c", "tail /etc/secret-data/*; sleep 600"]
        volumeMounts:
        - name: secret-volume
          mountPath: "/etc/secret-data"
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: my-password-vault

