# Jenkins CI/CD on Amazon EKS

Tài liệu này hướng dẫn triển khai Jenkins trên Amazon EKS, cấu hình Jenkins Agent trên EC2 và deploy ứng dụng lên Kubernetes.

---

## Bước 1. Khởi tạo môi trường VirtualBox (nếu cần)

Khởi động lại dịch vụ clipboard của VirtualBox:

```bash
killall VBoxClient
VBoxClient --clipboard
```

---

## Bước 2. Import tài nguyên IAM vào Terraform

Import IAM Role đã tồn tại vào Terraform state:

```bash
terraform import aws_iam_role.build DE00175-eks-build-role
```

---

## Bước 3. Cấu hình kết nối tới EKS

Cập nhật kubeconfig để truy cập cụm EKS:

```bash
aws eks update-kubeconfig \
  --region ap-southeast-1 \
  --name DE00175-eks
```

Kiểm tra kết nối:

```bash
kubectl get nodes
```

---

## Bước 4. Cài đặt NGINX Ingress Controller

Triển khai NGINX Ingress:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/aws/deploy.yaml
```

Chờ Ingress Controller sẵn sàng:

```bash
kubectl -n ingress-nginx wait \
  --for=condition=Available \
  deployment/ingress-nginx-controller \
  --timeout=180s
```

Kiểm tra:

```bash
kubectl get pods -n ingress-nginx
```

---

## Bước 5. Tạo StorageClass

Áp dụng StorageClass:

```bash
kubectl apply -f sc.yaml
```

---

## Bước 6. Triển khai Jenkins trên Kubernetes

### 6.1 Tạo namespace Jenkins

```bash
kubectl create namespace jenkins
```

### 6.2 Tạo Persistent Volume Claim

```bash
kubectl apply -f pvc.yaml
```

### 6.3 Deploy Jenkins

```bash
kubectl apply -f jenkins-deploy.yaml
```

Kiểm tra trạng thái:

```bash
kubectl get pods -n jenkins
kubectl get svc -n jenkins
```

---

## Bước 7. Lấy địa chỉ Jenkins

Lấy LoadBalancer của Jenkins:

```bash
kubectl -n jenkins get svc jenkins-service \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Ví dụ:

```
http://<jenkins-load-balancer>
```

---

## Bước 8. Lấy mật khẩu quản trị Jenkins

Lấy mật khẩu khởi tạo:

```bash
kubectl exec -it \
$(kubectl get pods -n jenkins -l app=jenkins -o name) \
-n jenkins \
-- cat /var/jenkins_home/secrets/initialAdminPassword
```

Dùng mật khẩu này để đăng nhập lần đầu vào Jenkins.

---

## Bước 9. Port Forward Jenkins (tùy chọn)

Nếu muốn truy cập Jenkins qua localhost:

```bash
kubectl port-forward svc/jenkins-service 8080:80 -n jenkins
```

Truy cập:

```
http://localhost:8080
```

---

## Bước 10. Kết nối tới EC2 Build Machine

SSH vào EC2:

```bash
ssh -i build-machine-key.pem ubuntu@13.212.232.100
```

---

## Bước 11. Đăng ký EC2 làm Jenkins Agent (JNLP)

### 11.1 Tạo Agent trên Jenkins

Trên Jenkins UI:

```
Manage Jenkins
→ Nodes
→ New Node
```

Thông tin cấu hình:

* Node name: `ec2-build`
* Type: `Permanent Agent`

Nhấn **Create**.

Cấu hình:

* Remote root directory:

```text
/home/ubuntu/jenkins
```

* Labels:

```text
ec2-build
```

* Launch method:

```text
Launch agent by connecting it to the controller
```

Nhấn **Save**.

---

### 11.2 Copy lệnh kết nối Agent

Vào node vừa tạo:

```
ec2-build
→ Launch agent
```

Copy lệnh Java có chứa:

* Secret
* Agent name

Ví dụ:

```bash
java -jar agent.jar \
  -url http://JENKINS_URL/ \
  -secret SECRET \
  -name worker \
  -webSocket \
  -workDir "/home/ubuntu/jenkins"
```

---

### 11.3 Chạy Jenkins Agent trên EC2

Tạo thư mục làm việc:

```bash
mkdir -p ~/jenkins
cd ~/jenkins
```

Khai báo URL Jenkins:

```bash
JENKINS_URL="http://ae4cf947f4bdb41f6908759212c14a5b-1885399582.ap-southeast-1.elb.amazonaws.com"
```

Tải agent.jar:

```bash
curl -sO \
http://a60511c66fc924da6ac555b848270fa5-2957345.ap-southeast-1.elb.amazonaws.com/jnlpJars/agent.jar
```

Khởi động Agent:

```bash
java -jar agent.jar \
  -url "${JENKINS_URL}/" \
  -secret f33fe6236370a9b4d3cf9f2738efd439f3f067aa3b6252bc6962964542c6301d \
  -name worker \
  -webSocket \
  -workDir "/home/ubuntu/jenkins"
```

Sau khi kết nối thành công, trạng thái node trên Jenkins sẽ chuyển sang:

```text
Connected
```

---

## Bước 12. Chuẩn bị biến môi trường Deploy

Lấy AWS Account ID:

```bash
ACCOUNT_ID=$(aws sts get-caller-identity \
  --query Account \
  --output text)
```

Khai báo Region:

```bash
REGION=ap-southeast-1
```

Khai báo ECR Repository:

```bash
REPO=$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/de00175-app
```

Lấy RDS Endpoint:

```bash
RDS_ENDPOINT=$(terraform output -raw mysql_endpoint)
```

---

## Bước 13. Đăng nhập ECR

```bash
aws ecr get-login-password \
  --region $REGION \
| docker login \
  --username AWS \
  --password-stdin $REPO
```

---

## Bước 14. Build và Push Docker Image

Build image:

```bash
docker build -t $REPO:latest .
```

Push image:

```bash
docker push $REPO:latest
```

---

## Bước 15. Tạo file Kubernetes Deployment cuối cùng

Thay thế các biến trong manifest:

```bash
sed \
  -e "s|REPLACE_WITH_RDS_ENDPOINT|$RDS_ENDPOINT|g" \
  -e 's|REPLACE_WITH_PASSWORD|Tai123456789|g' \
  -e "s|REPLACE_WITH_ECR_URI:latest|$REPO:latest|g" \
  app.yaml > app_final.yaml
```

---

## Bước 16. Deploy ứng dụng

Triển khai ứng dụng:

```bash
kubectl apply -f app_final.yaml --namespace=app
```

Kiểm tra:

```bash
kubectl get pods -n app
kubectl get svc -n app
kubectl get ingress -n app
```

---

## Bước 17. Kiểm tra ứng dụng

Lấy địa chỉ Ingress:

```bash
INGRESS=$(kubectl -n ingress-nginx get svc ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

Kiểm tra trang chủ:

```bash
curl -i http://$INGRESS/
```

Kết quả mong đợi:

```text
HTTP/1.1 200 OK
```

Kiểm tra API:

```bash
curl http://$INGRESS/api/items
```

Kết quả mong đợi:

```json
[]
```

Kiểm tra health endpoint:

```bash
curl http://$INGRESS/health
```

Hoặc mở trình duyệt:

```text
http://<INGRESS_HOSTNAME>/
http://<INGRESS_HOSTNAME>/health
```

---

## Kiến trúc triển khai

```text
Developer
    │
    ▼
 GitHub
    │
    ▼
 Jenkins Controller (EKS)
    │
    ▼
 Jenkins Agent (EC2)
    │
    ▼
 Docker Build
    │
    ▼
 Amazon ECR
    │
    ▼
 Amazon EKS
    │
    ▼
 Flask Application
    │
    ▼
 Amazon RDS
```

