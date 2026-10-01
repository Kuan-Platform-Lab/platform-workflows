# platform-workflows

Reusable GitHub Actions workflows dùng chung cho các service. Phase 3.

Xem Guide.md (Platform Engineering Lab) để biết bối cảnh.

## `java-service.yaml` (reusable)

Luồng: `test` (gitleaks + `./gradlew check`) → `image` (build amd64, Trivy HIGH/CRITICAL, SBOM) → `publish` (chỉ trên `main`: đa kiến trúc amd64+arm64 lên GHCR, tag `sha-<7 ký tự>`) → `update-gitops` (ghi tag mới vào `apps/<service>/values-dev.yaml` của platform-gitops).

Trên Pull Request chỉ chạy `test` + `image`. Check bắt buộc ở repo app: `ci / test`, `ci / image`.

Repo app chỉ cần:

```yaml
jobs:
  ci:
    uses: Kuan-Platform-Lab/platform-workflows/.github/workflows/java-service.yaml@main
    with: { service: order-service }
    secrets: inherit
```
kèm `permissions: { contents: read, packages: write }` ở cấp workflow.

### Secrets cần có (mỗi repo app)
- `GITOPS_TOKEN`: fine-grained PAT, chỉ repo `platform-gitops`, quyền Contents + Pull requests: Read and write. Thiếu token thì bước `update-gitops` chỉ cảnh báo và bỏ qua.
- `GITHUB_TOKEN` có sẵn, đủ để push GHCR.

### GHCR
Package mặc định **private**. Hoặc đặt package public (Package settings → Change visibility), hoặc tạo imagePullSecret `ghcr-pull` bằng classic PAT `read:packages`.
