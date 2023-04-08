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
```
