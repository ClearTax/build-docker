# Build DockerImage Action

A GitHub composite action that builds Docker images and pushes them to Amazon ECR (Elastic Container Registry).

## Description

This action streamlines the process of building Docker images and pushing them to Amazon ECR as part of your GitHub Actions workflow. It handles AWS authentication, ECR login, Docker Buildx setup, and the build/push process in a single reusable component.

## Features

- Automatic artifact download (optional)
- AWS credentials configuration
- Amazon ECR authentication
- Docker Buildx setup for efficient builds
- Docker image building with customizable parameters
- Automatic pushing to Amazon ECR
- Build caching for faster subsequent builds

## Inputs

### Required Inputs

| Input | Description |
|-------|-------------|
| `aws_access_key_id` | AWS Access Key ID for authentication |
| `aws_secret_access_key` | AWS Secret Access Key for authentication |
| `service` | Name of the service/repository in ECR |
| `dockerfile` | Path to the Dockerfile (default: `Dockerfile`) |
| `version` | Version tag for the Docker image |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `aws_region` | AWS Region where ECR is located | `ap-south-1` |
| `build_args` | Build arguments to pass to Docker build | `''` (empty) |
| `download_artifact` | Whether to download artifacts before building | `true` |
| `multi_arch` | Build multi-arch image (`linux/amd64,linux/arm64`) | `false` |
| `target_arch` | Desired architecture for your Docker image | `linux/amd64` |
| `context` | Docker build context path | `.` |
| `target` | Docker build target stage (for multi-stage builds) | `''` (empty) |
| `additional_tags` | Additional tags to push (newline-separated) | `''` (empty) |
| `secrets` | List of secrets to expose to the build (e.g., `key=string`) | |
| `secret-envs` | List of secret env vars to expose to the build (e.g., `key=envname`) | |
| `secret-files` | List of secret files to expose to the build (e.g., `key=filename`) | |

## Usage

```yaml
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      # Optional: Create and upload artifacts if needed
      - name: Create artifacts
        run: |
          # Commands to create artifacts
          
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: artifacts-${{github.run_id}}
          path: path/to/artifacts
      
      # Use the Build DockerImage action
      - name: Build and Push Docker Image
        uses: cleartax/build-docker@main  # Replace with your actual repository
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws_region: ap-south-1
          service: my-service
          dockerfile: ./path/to/Dockerfile
          version: ${{ github.sha }}
          build_args: |
            ARG1=value1
            ARG2=value2
          download_artifact: true
```

### With Build Secrets

You can pass secrets to the Docker build using three methods:

```yaml
      - name: Build and Push Docker Image
        uses: cleartax/build-docker@main
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          service: my-service
          dockerfile: Dockerfile
          version: ${{ github.sha }}
          # Inline secret values
          secrets: |
            GIT_AUTH_TOKEN=${{ secrets.GH_TOKEN }}
          # Expose environment variables as secrets
          secret-envs: |
            NPM_TOKEN
          # Mount secret files
          secret-files: |
            npmrc=${{ github.workspace }}/.npmrc
```

In your Dockerfile, access these secrets via `--mount=type=secret`:

```dockerfile
RUN --mount=type=secret,id=GIT_AUTH_TOKEN \
    export GIT_AUTH_TOKEN=$(cat /run/secrets/GIT_AUTH_TOKEN) && \
    git clone https://${GIT_AUTH_TOKEN}@github.com/org/repo.git

RUN --mount=type=secret,id=NPM_TOKEN \
    NPM_TOKEN=$(cat /run/secrets/NPM_TOKEN) npm install

RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm install
```

## How It Works

1. **Artifact Download** (Optional): If `download_artifact` is set to `true`, the action downloads artifacts from the current workflow run.

2. **AWS Configuration**: Configures AWS credentials for authentication with ECR.

3. **ECR Login**: Logs into Amazon ECR to allow pushing images.

4. **Docker Buildx Setup**: Sets up Docker Buildx for efficient multi-platform builds.

5. **Build and Push**: Builds the Docker image using the specified Dockerfile and pushes it to ECR with the provided tag.

## Caching

This action utilizes GitHub Actions cache for Docker layers, which significantly speeds up subsequent builds by reusing previously built layers.

## Notes

- Make sure your AWS credentials have appropriate permissions to push to ECR.
- The ECR repository must exist before using this action, or your AWS credentials must have permissions to create repositories.
- For security, always use GitHub Secrets to store your AWS credentials.
