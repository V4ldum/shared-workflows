# Shared Workflows

## Rust project

```yaml
name: CI/CD
on:
    push:
        branches: [main]
        paths:
            - "src/**"
            - "Dockerfile"
            - "docker-compose*.yml"

jobs:
    detect-changes:
        runs-on: ubuntu-latest
        outputs:
            src: ${{ steps.filter.outputs.src }}
            compose: ${{ steps.filter.outputs.compose }}
        steps:
            - uses: actions/checkout@v4
            - uses: dorny/paths-filter@v3
              id: filter
              with:
                  filters: |
                      src:
                          - 'src/**'
                          - 'Dockerfile'
                      compose:
                          - 'docker-compose*.yml'

    test:
        needs: detect-changes
        if: needs.detect-changes.outputs.src == 'true'
        uses: v4ldum/shared-workflows/.github/workflows/test-rust.yml@main

    build:
        needs: test
        uses: v4ldum/shared-workflows/.github/workflows/build.yml@main
        with:
            project: my-project
        secrets: inherit

    deploy:
        needs: [detect-changes, build]
        if: >-
            always() &&
            (needs.build.result == 'success' ||
             (needs.build.result == 'skipped' && needs.detect-changes.outputs.compose == 'true'))
        uses: v4ldum/shared-workflows/.github/workflows/deploy.yml@main
        with:
            project: my-project
        secrets: inherit

    cleanup:
        needs: build
        if: needs.build.result == 'success'
        uses: v4ldum/shared-workflows/.github/workflows/cleanup.yml@main
        with:
            project: my-project
        secrets: inherit
```
