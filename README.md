# Usage

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - https://github.com/diePartments/pimcore-k8s/base
  - https://github.com/diePartments/pimcore-k8s/overlay/persistence
  - https://github.com/diePartments/pimcore-k8s/overlay/initialization

helmCharts:
  # frontend deployment.
  - releaseName: pimcore-frontend
    name: pimcore-deployment
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
    repo: https://raw.githubusercontent.com/diePartments/pimcore-k8s/main/charts
    valuesInline:
      labels:
        tier: messenger
        role: async-worker
        app: pimcore
        lang: php
      fullname: messenger-async
      pimcore:
        selectorLabels:
          tier: messenger
          role: async-worker
          app: pimcore
          lang: php
        nginx:
          enabled: false
        php:
          probe: false
          command:
            - /bin/bash
            - -c
            - /var/www/bin/bin/console messenger:consume async -vv
```

NOTE: Due to limitations of kustomize, patches defined in remote layers have to be defined in local layers to be applied. 
