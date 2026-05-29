killall VBoxClient
VBoxClient --clipboard

aws eks update-kubeconfig --region ap-southeast-1 --name DE00175-eks
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/aws/deploy.yaml
kubectl -n ingress-nginx wait --for=condition=Available deployment/ingress-nginx-controller --timeout=180s
kubectl apply -f sc.yaml
kubectl create namespace jenkins
kubectl apply -f pvc.yaml
kubectl apply -f app.yaml --namespace=app --validate=false
kubectl apply -f jenkins-deploy.yaml
# Lấy địa chỉ Jenkins
kubectl -n jenkins get svc jenkins-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Lấy địa chỉ Nginx Ingress (App)
kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
kubectl exec -it $(kubectl get pods -n jenkins -l app=jenkins -o name) -n jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword

kubectl port-forward svc/jenkins-service 8080:80 -n jenkins
ssh -i build-machine-key.pem ubuntu@54.255.246.68

# Tải agent.jar và chạy (paste lệnh từ Jenkins UI)
mkdir -p ~/jenkins && cd ~/jenkins
JENKINS_URL="http://a93be265b60474cf2a0d2118c9000751-304917590.ap-southeast-1.elb.amazonaws.com"

java -jar agent.jar -url "${JENKINS_URL}/" -secret 12ef79d71305e4439074ab288fd6817219094af987586f058c2d6b4447a2ab7a -name worker -webSocket -workDir "/home/ubuntu/jenkins"


# 1. ACCOUNT_ID: Giữ nguyên nếu bạn đang ở đúng tài khoản đó
ACCOUNT_ID=891920435433

# 2. REGION: Giữ nguyên
REGION=ap-southeast-1

# 3. REPO: Phải khớp với tên repo trên ECR của bạn
# Trong log lỗi ban đầu bạn để là: 891920435433.dkr.ecr.ap-southeast-1.amazonaws.com/de00175-app
REPO=$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/de00175-app

# 4. PATH Terraform: Kiểm tra xem thư mục chứa file .tf của bạn là gì
# Nếu bạn đang để file main.tf ở thư mục gốc hoặc tên khác, hãy sửa lại cho đúng
RDS_ENDPOINT=$(terraform output -raw mysql_endpoint)
sed \
  -e "s|REPLACE_WITH_RDS_ENDPOINT|$RDS_ENDPOINT|g" \
  -e 's|REPLACE_WITH_PASSWORD|Tai123456789|g' \
  -e "s|REPLACE_WITH_ECR_URI:latest|$REPO:v1|g" \
  app.yaml > app_final.yaml

# Sau đó apply file đã được thay thế
kubectl apply -f app_final.yaml --namespace=app
