* Deployment Yaml *

  apiVersion: platform.confluent.io/v1beta1
kind: FlinkApplication
metadata:
  name: otp-process-sync-in-nonmon
  namespace: ot-dev-app
spec:
  flinkEnvironment: flink-env-dev
  image: org/is/otp-sync-process-servicing-order-sync-publisher:2.0.151
  flinkVersion: v1_19
  flinkConfiguration:
    taskmanager.numberOfTaskSlots: "2"
    classloader.resolve-order: parent-first
    env.java.opts.all: --add-exports=java.base/sun.net.util=ALL-UNNAMED
      --add-exports=java.rmi/sun.rmi.registry=ALL-UNNAMED
      --add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED
      --add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED
      --add-exports=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED
      --add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED
      --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED
      --add-exports=java.security.jgss/sun.security.krb5=ALL-UNNAMED
      --add-opens=java.base/java.lang=ALL-UNNAMED
      --add-opens=java.base/java.net=ALL-UNNAMED
      --add-opens=java.base/java.io=ALL-UNNAMED
      --add-opens=java.base/java.nio=ALL-UNNAMED
      --add-opens=java.base/sun.nio.ch=ALL-UNNAMED
      --add-opens=java.base/java.lang.reflect=ALL-UNNAMED
      --add-opens=java.base/java.text=ALL-UNNAMED
      --add-opens=java.base/java.time=ALL-UNNAMED
      --add-opens=java.base/java.util=ALL-UNNAMED
      --add-opens=java.base/java.util.concurrent=ALL-UNNAMED
      --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED
      --add-opens=java.base/java.util.concurrent.locks=ALL-UNNAMED
  serviceAccount: flink
  jobManager:
    resource:
      memory: 4096m
      cpu: 1
    podTemplate:
      spec:
        containers:
          - name: flink-main-container
            env:
              - name: KAFKA_SSL_PWD
                valueFrom:
                  secretKeyRef:
                    name: kafka-secret
                    key: KAFKA_SSL_PWD
              - name: MONGO_PWD
                valueFrom:
                  secretKeyRef:
                    name: mongodb-secret
                    key: MONGO_PWD
            volumeMounts:
              - mountPath: /mnt/flink
                name: flink-persistent-storage
              - mountPath: /data
                name: flink-otp-storage
              - mountPath: /work/application.properties
                name: otp-sync-pso-nonmon-publisher-default-configmap
                readOnly: true
                subPath: application.properties
              - mountPath: /work/flink_config.properties
                name: otp-sync-pso-nonmon-publisher-default-configmap
                readOnly: true
                subPath: flink_config.properties
              - mountPath: /work/certs
                name: otp-sync-pso-nonmon-publisher-default-secrets
                readOnly: true
        volumes:
          - name: flink-persistent-storage
            persistentVolumeClaim:
              claimName: flink-root-claim
          - name: flink-otp-storage
            persistentVolumeClaim:
              claimName: flink-nfs-storage
          - name: otp-sync-pso-nonmon-publisher-default-configmap
            configMap:
              name: otp-sync-pso-nonmon-publisher-default-configmap
          - name: otp-sync-pso-nonmon-publisher-default-secrets
            secret:
              secretName: otp-sync-pso-nonmon-publisher-default-secrets
  taskManager:
    resource:
      memory: 8192m
      cpu: 1
    podTemplate:
      spec:
        containers:
          - name: flink-main-container
            env:
              - name: KAFKA_SSL_PWD
                valueFrom:
                  secretKeyRef:
                    name: kafka-secret
                    key: KAFKA_SSL_PWD
              - name: MONGO_PWD
                valueFrom:
                  secretKeyRef:
                    name: mongodb-secret
                    key: MONGO_PWD
            volumeMounts:
              - mountPath: /mnt/flink
                name: flink-persistent-storage
              - mountPath: /data
                name: flink-otp-storage
              - mountPath: /work/application.properties
                name: otp-sync-pso-nonmon-publisher-default-configmap
                readOnly: true
                subPath: application.properties
              - mountPath: /work/flink_config.properties
                name: otp-sync-pso-nonmon-publisher-default-configmap
                readOnly: true
                subPath: flink_config.properties
              - mountPath: /work/certs
                name: otp-sync-pso-nonmon-publisher-default-secrets
                readOnly: true
        volumes:
          - name: flink-persistent-storage
            persistentVolumeClaim:
              claimName: flink-root-claim
          - name: flink-otp-storage
            persistentVolumeClaim:
              claimName: flink-nfs-storage
          - name: otp-sync-pso-nonmon-publisher-default-configmap
            configMap:
              name: otp-sync-pso-nonmon-publisher-default-configmap
          - name: otp-sync-pso-nonmon-publisher-default-secrets
            secret:
              secretName: otp-sync-pso-nonmon-publisher-default-secrets
  job:
    jarURI: local:///work/otp-sync-process-servicing-order-sync-publisher-2.0.151.jar
    parallelism: 4
    args: ["2025-07-14"]
    upgradeMode: stateless
  cmfRestClassRef:
    name: default
    namespace: confluent-operators
