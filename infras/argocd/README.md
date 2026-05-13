# Argo CD (Helm)

Thư mục này chứa file values và manifest bổ trợ để cài [Argo CD](https://argo-cd.readthedocs.io/) lên Kubernetes. Bạn có thể cài trực tiếp từ Helm repository của dự án argo-helm (giống các service khác trong repo), hoặc dùng chart đã vendor trong `argo-cd-3.4.2/` nếu bạn đang giữ bản chart cố định trong Git.

## Cài đặt từ Helm repository

Chạy từ thư mục gốc của repository `cicd` (để đường dẫn `-f` đúng).

```sh
helm repo add argo https://argoproj.github.io/argo-helm

helm repo update

helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  --create-namespace \
  -f infras/argocd/values-dev.yaml \
  -f infras/argocd/values-prod.yaml
```

- `helm repo add`: đăng ký repository chứa chart `argo-cd`.
- `helm repo update`: tải metadata mới nhất từ các repo đã add (nên chạy trước khi cài hoặc đổi version).
- `--create-namespace`: tạo namespace `argocd` nếu chưa có.

Nếu bạn muốn ghim phiên bản chart thay vì dùng bản mới nhất, thêm ví dụ `--version 8.6.0` (chọn version tương thích với cluster và tài liệu [argo-helm](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd)).

## Cài đặt từ chart vendored trong repo (tùy chọn)

Khi thư mục `argo-cd-3.4.2/` là nguồn chart thật sự (đủ `Chart.yaml` và dependency), có thể cài local thay vì pull từ repo:

```sh
cd infras/argocd

helm dependency build ./argo-cd-3.4.2

helm upgrade --install argocd ./argo-cd-3.4.2 \
  -n argocd \
  --create-namespace \
  -f values-dev.yaml \
  -f values-prod.yaml
```

## Namespace `argocd`

`--create-namespace` trong lệnh Helm là đủ cho lần cài đầu. Nếu sau này bạn thêm manifest `namespace.yaml` (label, annotation, quota, …), có thể `kubectl apply` trước bước Helm.

## Ứng dụng bootstrap `root` (app-of-apps)

Ứng dụng `root` trỏ Git tới thư mục `apps/` trên repo của bạn. Chỉ nên dùng **một** cách để tránh hai `Application` cùng tên `root`:

- **Values (prod):** `values-prod.yaml` khai báo `extraObjects` để Helm render `Application` khi cài hoặc upgrade.
- **Manifest:** sau khi Argo CD chạy, `kubectl apply -f infras/argocd/main.yaml`. Khi chọn cách này, bỏ phần `root` trong `extraObjects` của values.

Nhớ chỉnh `repoURL`, `targetRevision`, và `path` cho đúng môi trường.

## Cấu trúc thư mục

| Đường dẫn | Mô tả |
|-----------|--------|
| `argo-cd-3.4.2/` | Chart Argo CD vendored (dùng khi cài local) |
| `values-dev.yaml` | Values mặc định / dùng chung |
| `values-prod.yaml` | Ghi đè prod (ví dụ `global.domain`, `extraObjects` cho `root`) |
| `values.yaml` | Values bổ sung trong thư mục này (ghép thêm bằng `-f` nếu cần) |
| `main.yaml` | `Application` `root` độc lập (tùy chọn, nếu không dùng `extraObjects`) |

## Tài liệu tham khảo

- [Argo CD operator manual](https://argo-cd.readthedocs.io/en/stable/operator-manual/)
- [Chart argo-cd (argo-helm)](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd)
