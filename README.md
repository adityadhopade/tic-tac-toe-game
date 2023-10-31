<h2 align="center">CICD using GitHub Action TIC-TAC-TOE-GAME</h2>

### Workflow of the Tic-Tac-Toe Application
![TIC_TAC_TOE_using_GITHUB](https://github.com/adityadhopade/tic-tac-toe-game/assets/48392204/9a86367e-1331-460a-9c8d-037c60c48c4b)

### Tools Implemented
- Git
- GitHub Actions (Adding the Github Secrets)
- Self-Hosted Runner (Creation of Self Hosted Runner)
- EC2 Instance (Ubuntu 22.04 t2 medium 20GB Storage)
- Terraform (to build EKS)
- Installing AWS CLI for using eksctl
- Sonarqube (Configured via Docker)
- Install npm
- Trivy (Used for File Scanning and Trivy Image Scanning)
- Elastic Kubernetes Serice [E.K.S]
- Slack Integration to notify users about the status of the deployment

### Mandatory ports to keep open
- 3000 (for app to run)
- 9000 (for sonarqube dashboard)
- 22, 443, 80
- 3**** (Generated randomly while creating the EKS Cluster)

### Mandatory Installations on the EC2 Instances 
```
#!/bin/bash
sudo apt update -y


sudo apt install docker.io -y
sudo usermod -aG docker username
newgrp docker
sudo chmod 777 /var/run/docker.sock

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
 
sudo touch /etc/apt/keyrings/adoptium.asc
sudo wget -O /etc/apt/keyrings/adoptium.asc https://packages.adoptium.net/artifactory/api/gpg/key/public
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version

# Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

# Install Terraform
sudo apt install wget -y
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Install kubectl
sudo apt update
sudo apt install curl -y
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install AWS CLI 
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install unzip -y
unzip awscliv2.zip
sudo ./aws/install

# Install Node.js 16 and npm
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/nodesource-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/nodesource-archive-keyring.gpg] https://deb.nodesource.com/node_16.x focal main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update
sudo apt install -y nodejs
```

### Check for all of the installations installed above

```
docker --version
trivy --version
terraform --version
aws --version
kubectl version
node -v
java --version
```
  
### TO make the changes in the Terraform files

```
git clone https://github.com/adityadhopade/tic-tac-toe-game.git
cd TIC-TAC-TOE
cd Eks-terraform
```
![image](https://github.com/adityadhopade/tic-tac-toe-game/assets/48392204/3c86f64f-3d31-4efc-94f7-fa5af44df0ce)

![image](https://github.com/adityadhopade/tic-tac-toe-game/assets/48392204/3397b011-14f9-45b7-8b48-13ced4606f87)

### Deploy to EKS
```
 - name: Update kubeconfig
   run: aws eks --region <cluster-region> update-kubeconfig --name <cluster-name>
```
```
  - name: Deploy to K8's
    run: |
      # Delete the existing deployment if it exists
      if kubectl get deployment tic-tac-toe &>/dev/null; then
        kubectl delete deployment tic-tac-toe
        echo "Existing deployment removed"
      fi


      # Delete the existing service if it exists
      if kubectl get service tic-tac-toe-service &>/dev/null; then
        kubectl delete service tic-tac-toe-service
        echo "Existing service removed"
      fi

      kubectl apply -f deployment-service.yml 

      current_directory=$(pwd)
      echo "Current directory is: $current_directory"
```

### After implementation just remember to delete all the resources created
```
Delete ec2 instance
Delete iam role
Delete github actions secrets
```

### For Slack Notification
```
name: Send a Slack Notification
  if: always() # as we want it to run even if our job fails
  uses: act10ns/slack@v1
  with:
    status: ${{ job.status }}
    steps: ${{ toJson(steps) }}
    channel: '#githubactions-eks' # your created Channel
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Game Image
![image](https://github.com/adityadhopade/tic-tac-toe-game/assets/48392204/0f69eeec-6768-4dd2-98d4-185b3c3251e4)


