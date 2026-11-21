# Shared Workflows

## Rust CD

```yaml
name: Build & Deploy
on:
    workflow_dispatch:
    push:
        branches:
            - "main"
        paths:
            - src/**
            - Dockerfile
            - kubernetes.yml

concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true

jobs:
    detect-changes:
        runs-on: ubuntu-latest
        permissions:
            contents: read
        outputs:
            build: ${{ steps.filter.outputs.build }}
            deploy: ${{ steps.filter.outputs.deploy }}
        steps:
            - uses: actions/checkout@v7
            - uses: dorny/paths-filter@v4
              id: filter
              with:
                  filters: |
                      build:
                          - 'src/**'
                          - 'Dockerfile'
                      deploy:
                          - 'kubernetes.yml'

    test:
        needs: detect-changes
        if: needs.detect-changes.outputs.build == 'true'
        uses: v4ldum/shared-workflows/.github/workflows/test-rust.yml@main
        with:
            format: false
        permissions:
            contents: read

    build:
        needs: test
        uses: v4ldum/shared-workflows/.github/workflows/build.yml@main
        secrets: inherit
        with:
            project: PROJECT
        permissions:
            contents: read
            packages: write

    deploy:
        needs: [detect-changes, build]
        if: >-
            !cancelled() && !failure() &&
            (needs.build.result == 'success' ||
              needs.build.result == 'skipped' &&
                needs.detect-changes.outputs.deploy == 'true')
        uses: v4ldum/shared-workflows/.github/workflows/deploy.yml@main
        secrets: inherit
        with:
            project: PROJECT
            apply: ${{ needs.detect-changes.outputs.deploy == 'true' }}
            rollout: ${{ needs.detect-changes.outputs.build == 'true' }}
        permissions:
            contents: read
            id-token: write

    cleanup:
        needs: [build, deploy]
        if: needs.build.result == 'success'
        uses: v4ldum/shared-workflows/.github/workflows/cleanup.yml@main
        secrets: inherit
        with:
            project: PROJECT
        permissions:
            packages: write

```

## Flutter CD

```yaml
name: Build & Deploy
on:
    workflow_dispatch:
    push:
        branches:
            - "main"
        paths:
            - lib/**
            - Dockerfile
            - kubernetes.yml

concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true
    
jobs:
    detect-changes:
        runs-on: ubuntu-latest
        permissions:
            contents: read
        outputs:
            build: ${{ steps.filter.outputs.build }}
            deploy: ${{ steps.filter.outputs.deploy }}
        steps:
            - uses: actions/checkout@v7
            - uses: dorny/paths-filter@v4
              id: filter
              with:
                  filters: |
                      build:
                          - 'lib/**'
                          - 'Dockerfile'
                      deploy:
                          - 'kubernetes.yml'

    test:
        needs: detect-changes
        if: needs.detect-changes.outputs.build == 'true'
        uses: v4ldum/shared-workflows/.github/workflows/test-flutter.yml@main
        secrets: inherit
        with:
            format: false
            test: false
        permissions:
            contents: read

    build:
        needs: test
        uses: v4ldum/shared-workflows/.github/workflows/build.yml@main
        secrets: inherit
        with:
            project: PROJECT
        permissions:
            contents: read
            packages: write

    deploy:
        needs: [detect-changes, build]
        if: >-
            !cancelled() && !failure() &&
            (needs.build.result == 'success' ||
              needs.build.result == 'skipped' &&
                needs.detect-changes.outputs.deploy == 'true')
        uses: v4ldum/shared-workflows/.github/workflows/deploy.yml@main
        secrets: inherit
        with:
            project: PROJECT
            apply: ${{ needs.detect-changes.outputs.deploy == 'true' }}
            rollout: ${{ needs.detect-changes.outputs.build == 'true' }}
        permissions:
            contents: read
            id-token: write

    cleanup:
        needs: [build, deploy]
        if: needs.build.result == 'success'
        uses: v4ldum/shared-workflows/.github/workflows/cleanup.yml@main
        secrets: inherit
        with:
            project: PROJECT
        permissions:
            packages: write

```

## Static Shock CD (legacy)

```yaml
name: Build & Deploy
on:
    workflow_dispatch:
    push:
        branches:
            - "main"
        paths:
            - bin/**
            - source/**
            - Dockerfile
            - kubernetes.yml

concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true

jobs:
    detect-changes:
        runs-on: ubuntu-latest
        permissions:
            contents: read
        outputs:
            build: ${{ steps.filter.outputs.build }}
            deploy: ${{ steps.filter.outputs.deploy }}
        steps:
            - uses: actions/checkout@v7
            - uses: dorny/paths-filter@v4
              id: filter
              with:
                  filters: |
                      build:
                          - 'bin/**'
                          - 'source/**'
                          - 'Dockerfile'
                      deploy:
                          - 'kubernetes.yml'

    test:
        needs: detect-changes
        if: needs.detect-changes.outputs.build == 'true'
        uses: v4ldum/shared-workflows/.github/workflows/test-static-shock.yml@main
        with:
            enabled: true
        permissions:
            contents: read

    build:
        needs: test
        uses: v4ldum/shared-workflows/.github/workflows/build.yml@main
        secrets: inherit
        with:
            project: PROJECT
        permissions:
            contents: read
            packages: write

    deploy:
        needs: [detect-changes, build]
        if: >-
            !cancelled() && !failure() &&
            (needs.build.result == 'success' ||
              needs.build.result == 'skipped' &&
                needs.detect-changes.outputs.deploy == 'true')
        uses: v4ldum/shared-workflows/.github/workflows/deploy.yml@main
        secrets: inherit
        with:
            project: PROJECT
            apply: ${{ needs.detect-changes.outputs.deploy == 'true' }}
            rollout: ${{ needs.detect-changes.outputs.build == 'true' }}
        permissions:
            contents: read
            id-token: write

    cleanup:
        needs: [build, deploy]
        if: needs.build.result == 'success'
        uses: v4ldum/shared-workflows/.github/workflows/cleanup.yml@main
        secrets: inherit
        with:
            project: PROJECT
        permissions:
            packages: write
```

## Rust Audit Dependencies

```yaml
name: Security audit
on:
    workflow_dispatch:
    schedule:
        - cron: "52 4 10 * *"
          timezone: "Europe/Paris"

jobs:
    audit:
        uses: v4ldum/shared-workflows/.github/workflows/audit-rust.yml@main
```
