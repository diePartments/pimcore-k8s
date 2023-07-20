# Usage

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

helmCharts:
  # frontend deployment.
  - releaseName: pimcore-frontend
    name: pimcore-deployment
    repo: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/charts
    valuesInline:
      fullname: pimcore-frontend
      labels:
        tier: frontend
        role: web
        app: pimcore
        lang: php
      pimcore:
        selectorLabels:
          tier: frontend
          role: web
          app: pimcore
          lang: php
        php:
          command:
            - /bin/bash
          args:
            - -c
            - /var/www/bin/console cache:warmup & php-fpm -F

  # messenger deployments.
  - releaseName: messenger-async
    name: pimcore-deployment
    repo: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/charts
    valuesInline:
      labels:
        tier: messenger
        app: pimcore
        role: async-worker
        lang: php
      fullname: messenger-async
      pimcore:
        selectorLabels:
          tier: messenger
          app: pimcore
          role: async-worker
          lang: php
        nginx:
          enabled: false
        php:
          probe: false
          command:
            - /bin/bash
            - -c
            - /var/www/bin/bin/console messenger:consume async -vv

resources:
  - https://github.com/diePartments/pimcore-k8s//base?ref=main
  - https://github.com/diePartments/pimcore-k8s//persistence?ref=main
  - https://github.com/diePartments/pimcore-k8s//base?ref=main

patches:
  # Attach mandatory envs from configmaps and secrets
  - target:
      kind: Deployment
      labelSelector: app=pimcore
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/containers/pimcore-env-from.yaml
  - target:
      kind: CronJob
      labelSelector: app=pimcore
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/containers/cron-pimcore-env-from.yaml

  # Attach regred to all deployments and cronjobs
  - target:
      kind: Deployment
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/deployment-pull-secret.yaml
  - target:
      kind: CronJob
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/cronjob-pull-secret.yaml

  # Wait for pimcore backend deployments
  - target:
      kind: Deployment
      labelSelector: tier!=ops,app=pimcore,primary!=true,!primary
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/init-containers/wait-for-pimcore.yaml

  # Attach php-conf-d
  - target:
      kind: Deployment
      labelSelector: app=pimcore,lang=php
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/volumes/mount-php-conf-d.yaml
  - target:
      kind: CronJob
      labelSelector: app=pimcore,lang=php
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/volumes/cron-mount-php-conf-d.yaml

  # Attach php-fpm-d
  - target:
      kind: Deployment
      labelSelector: app=pimcore,role=web,lang=php
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/volumes/mount-php-fpm-d.yaml

  # Attach nginx-conf-d
  - target:
      kind: Deployment
      labelSelector: role=web,app=pimcore
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/volumes/mount-nginx-conf-d.yaml

  # Add php-fpm exporter
  - target:
      kind: Deployment
      labelSelector: role=web,lang=php
    path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/base/patches/containers/php-fpm-exporter.yaml

    # Attach copy public init container
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/initialization/patches/init-containers/copy-public.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,role=web,primary=true

  # Attach create assets init container
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/initialization/patches/init-containers/create-assets.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,role=web,primary=true

  # Attach deploy-layouts init container
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/initialization/patches/init-containers/deploy-layouts.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,primary=true

  # Attach create-classes init container
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/initialization/patches/init-containers/create-classes.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,primary=true

  # Attach db-migrations init container
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/initialization/patches/init-containers/db-migrations.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,primary=true

  # Chown for www-data
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/initialization/patches/init-containers/chown-www-data.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,role in (web,async-worker)

  # Attach pimcore volumes to deployments
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/pimcore-volumes.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/pimcore-volumes-cron.yaml
    target:
      kind: CronJob
      labelSelector: app=pimcore
  # Mount pimcore volumes.
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/mount-pimcore-volumes.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/mount-pimcore-volumes-cron.yaml
    target:
      kind: Cron
      labelSelector: app=pimcore
  # Mount Pimcore volumes to nginx containers
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/mount-pimcore-volumes-nginx.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore,role=web

  # Attach symfony cache volume
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/pimcore-cache.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore
  # Mount symfony cache
  - path: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/persistence/patches/volumes/mount-pimcore-cache.yaml
    target:
      kind: Deployment
      labelSelector: app=pimcore
```

NOTE: Due to limitations of kustomize, patches have to be defined in local layers to be applied.
