# build-push-to-ecr

Github action responsible for building and pushing docker images to ECR registries following a specific
convention

## Conventions

This action assumes that there's an artifact created in a previous step with a series of specific files:

- `docker/Dockerfile`: Docker file definition
- `docker/ecosystem.config.js`: PM2 application execution config
- `dist/index.js`: Application binary

## Inputs

- `name`: (**required**) Name of the docker image
- `version`: (**required**) Version to be published. Every time an image is published is also tagged as `latest`
- `from-artifact`: (**required**) the Github artifact to download to get the binaries and `Dockerfile` from.
- `aws-region`: (**required**) AWS region where the ECR is located.
- `aws-role-to-assume`: (**required**) AWS role ARN allowed to push images to ECR.

## Usage

In this example we're building a `griddo-api-prv:1.0.0` image and pushing it to a certain ECR registry:

```yaml
    griddo-build:
        runs-on: ubuntu-latest
        - name: Publish Private API Artifacts
            uses: actions/upload-artifact@v3
            with:
            name: griddo-api
            path: |
                packages/griddo-api/dist/index.js
                packages/griddo-api/docker/Dockerfile
                packages/griddo-api/docker/ecosystem.config.js
            retention-days: 1

    griddo-api-prv-docker:
        runs-on: ubuntu-latest
        needs:
            - Publish
        steps:
            - uses: Secuoyas-Experience/build-push-docker-to-ecr@v1
            with:
                name: griddo-api-prv
                version: 1.0.0
                from-artifact: griddo-api
                aws-region: ${{ secrets.AWS_REGION }}
                aws-role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME_ARN }}
```

Notice that the AWS information must come from Github secrets to avoid sensitive information leaking.
