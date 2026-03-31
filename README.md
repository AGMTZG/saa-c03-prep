## Local Practice Setup

Practice AWS commands locally using [LocalStack](https://localstack.io) — no AWS account or real credentials needed.

---

### 1. Run LocalStack

**Option A — Docker (quick start)**
```bash
docker pull localstack/localstack
docker run --rm -it -p 4566:4566 -p 4571:4571 localstack/localstack
```
> Add `-d` to run in the background and free up the terminal.

**Option B — Docker Compose (recommended)**
```yaml
services:
  localstack:
    image: localstack/localstack
    container_name: localstack_main
    ports:
      - "4566:4566"
      - "4571:4571"
    environment:
      - SERVICES=s3,dynamodb,lambda
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
    volumes:
      - "./localstack-data:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```
```bash
docker compose up -d
```

---

### 2. Install AWS CLI
```bash
sudo apt update && sudo apt install -y unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

---

### 3. Configure environment variables

Add the following to your `~/.bashrc` (or `~/.zshrc`):
```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
export AWS_ENDPOINT_URL=http://localhost:4566
```

Apply changes:
```bash
source ~/.bashrc
```

> These are dummy credentials — LocalStack doesn't validate them.
> `AWS_ENDPOINT_URL` redirects all CLI calls to your local instance instead of real AWS.

---

### 4. Verify everything works
```bash
aws s3 mb s3://test-bucket
aws s3 ls
```

If you see `test-bucket` listed, LocalStack is running correctly.
