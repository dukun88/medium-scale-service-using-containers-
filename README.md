# Diagram Arsitektur
![Selection_004](https://github.com/user-attachments/assets/6f381469-fc62-450d-85b3-b8f7bd2dc031)

## 1. Memeriksa VPC
![Selection_005](https://github.com/user-attachments/assets/e2a28622-488b-4284-83b1-f6d0fb9d20a8)
- Buka AWS Console
- Cari VPC Service
- Check pada isw-vpc
- Jika Sudah sesuai Lanjut ke tahap berikutnya

## 2. Membuat ECR Repository
![Selection_007](https://github.com/user-attachments/assets/94b19924-6f6c-4a1c-af0e-83038a3128dd)

### Membuat Repo ECR Frontend & Backend
- Buka dan login ke Console AWS
- Cari ECR Service
- "Create Repository"
  
    **General setting**

  Visibillity settings = Private

  (isikan nama poada kolom kosong yang tersedia) = fc_frontend

  *biarkan settings Default konfig yg lain*

  "Create Repository"

***Ulangi cara tersebut untuk  membuat repo backend***

### Membuat resource Cloud9
- Cari Clound9 service
- "Create Environtment"
  
    **Details**

  Name = my-cloud9

  Description = my-cloud9

  Environtment type = New EC2 Instance

    **New EC2 Instance**

  Instance type = t2.micro

  Platform = Amazon Linux

  Timeout = 30 minutes

    **Network Setting**

  *biarkan dengan settingan default*

  "Create"

- Coba buka Cloud9 yang sudah kita buat

## 3. Docker Build dan Push ke ECR

### Clone source code

- Frontend
  
  ```
  git clone https://github.com/iboen/fc_frontend
  ```

- Backend

  ```
  git clone https://github.com/iboen/fc_backend
  ```

### Build dan push image ke ECR

- Buka service Cloud9
- Masuk ke folder frontend

  ```
  cd frontend
  ```
  
- Buka Service ECR di console AWS
- Pilih repo fc-frontend
- "View Push Commands"
- Ikuti alur command push yang tertera


  ![Selection_008](https://github.com/user-attachments/assets/6bea35d2-c717-4507-89a8-b548748f0138)

  
- Check Kembali pada Service ECR apakah image tersebut sudah berada di Repository

  ***Ulangi tahapan tersebut untuk build dan push image backend***

## 4. Membuat Security Group

![Selection_009](https://github.com/user-attachments/assets/3d10fea2-ee99-4824-8f7e-96e475a8a433)

- Cari service security gruop pada Console AWS
- "Create Security group"
  
  
    **ECS ALB Security group**
  
      Name : my-ecs-alb-sg
  
      Description : my-ecs-alb-sg
  
      VPC : isw-vpc
  
      *Inbound rules*
  
      Add rules according to the diagram :
  
      1.Type : HTTP, Port : 80, Source : Anywhere IPv4 (0.0.0.0/0)
  
      2.Type : HTTPS, Port : 443, Source : Anywhwre IPv4 (0.0.0.0/0)
  
      *Outbound rules*
  
      Default (0.0.0.0/0)
  
      Add tag Name : my-ecs-alb-sg
  
      "Create Security Group"
  
  
  
    **ECS Security group**
  
      Name : my-ecs-sg
  
      Description : my-ecs-sg
  
      VPC : isw-vpc
  
      *Inbound rules*
  
      Add rules according to the diagram :
  
      Type : All TCP, Source :  my-ecs-alb-sg
  
      *Outbound rules*
  
      Default (0.0.0.0/0)
  
      Add tag Name : my-ecs-sg
  
      "Create Security Group"
  
  
  
    **RDS Security group**
  
      Name : my-ecs-rds-sg
  
      Description : my-ecs-rds-sg
  
      VPC : isw-vpc
  
      *Inbound rules*
  
      Add rules according to the diagram :
  
      1.Type : MYSQL, Port : 3306, Source : my-openvpn-sg
  
      2.Type : MYSQL, Port : 3306, Source : my-ecs-sg
  
      *Outbound rules*
  
      Default (0.0.0.0/0)
  
      Add tag Name : my-ecs-rds-sg
  
      "Create Security Group"

- Untuk security group open vpn kita bisa memakai yang sebelumnya sudah kita buat

## 5. Membuat ELB dan menyiapkan Route53

![Selection_010](https://github.com/user-attachments/assets/12de1436-03ec-4dd6-8412-3b36bd308fb1)

### Membuat target group

  - Cari target group pada AWS Console
  - "Create target Gruop"

    **Basic configuration**

    Chose target type = Instance

    Target group name = my-ecs-frontend-80

    Protocol - Port = HTTP (80)

    IP Address type = IPv4

    VPC = isw-vpc

    Protocol version = Default

    **Health checks**

    *Biarkan Default atau berada di slash (/)*

    "Next"

    *Karna tidak menggunakan ec2 kita bisa langsung create*

    "Create"

- Ulangi langkah tadi untuk membuat target gruop backend dengan atribut

    Target group name = my-ecs-backend-8080

    Protocol - Port = HTTP (8080)

### Membuat loadbalancers

- Cari Loadbalancers pada Console AWS
- "Create Load Balancer"
- Pilih "Application load balancers"

    **Basic configuration**

    Load Balancer name = my-ecs-lb

    Scheme = Internet-facing

    IP Address type = IPv4

    **Network Mapping**

    VPC = isw-vpc

    Pilih kedua Mappingnya

    **Security Group**

    Hapus default dan pilih = ecs-alb-sg

    **Listeners And Routing**

    Untuk Port 80 kita akan masukan Frontend pada Defaulkt action

    add listeners untuk Https (443) dan masukan frontend juga pada default action

    **Secure Listeners Setting**

    Kita gunakan sertifikat SSL yang sudah dibuat dengan ACM

   "Create"

- Edit Rules Listeners 80 agar forward ke port 443 (HTTPS)
- Add rule pada listeners 443 jika user mengakses "api" buat ia diarahkan ke Target group backend(8080)

### Arahakan Subdoamin ke Loadbalancer

- Cari Route53 pada Console AWS
- Masuk ke DNS
- Edit domain yg akan kita pakai
- pilih sub domain "api"
- Arahakan ke loadbalancer yang sudah kita buat
- Buat subdomain baru untuk frontend
- "Create record"
  
    **Quick record name**

    Record name = ecs

    Turn on Alias

    Record Traffic alias = Alias to application load balancers

    Region = Jakarta

    Loadbalancers = my-ecs-lb

    "Create Records"

## 6. Memeriksa dan Mendapatkan informasi RDS

![Selection_012](https://github.com/user-attachments/assets/54599d5f-8de6-4a49-ae6a-fb2dab65c1c8)

- Cari RDS Service pada Console AWS
- Pilih RDS yang akan di gunakan
- Check pada Bagian Conectivity dan Security

![Selection_013](https://github.com/user-attachments/assets/7779d8d2-0d50-40e7-9f97-602c6413ff75)

- Pastikan Vpc dan Security groupnya sudah sesuai
- Pastikan juga Endpoint sudah bisa di pakai dan di aplikasikan

## 7. Membuat ECS Cluster

![Selection_014](https://github.com/user-attachments/assets/1f49d513-f94f-4404-b793-a00f5e8d5a9b)

- Cari ECS pada Console AWS
- "Create cluster"

    **Cluister Configuration**

    Cluster Name = my-ecs-cluster

    **Infrastructure**

    Pilih "Amazon EC2 Instances"

    Auto Scaling Group (ASG) = Cretae new ASG

    Provisionig model = On-demand

    Container instance AMI = Amazon linux 2

    EC2 Instance type = c5.Large

    EC2 INstance role = default

    Desained capacity = default

    SSH key pair = gunakan key yang pernah kita buat untuk akses menggunakan ssh

    Root EBS volume size = default

    **Network settings**

    VPC = isw-vpc

    Subnets = isw-private-subnet-app-c

    Security group = use existing

    Security group name = my-ecs-sg

    Auto assign public IP = Use subnet setting

    "Create"

## 8. Membuat Task Definition di ECS

![Selection_015](https://github.com/user-attachments/assets/e754c2c3-0c9c-48ca-935c-e63e5cda837b)

- Cari ECS pada console AWS
- Pilih menu "Task Definition"
- "Create New task definition"

    **Task definition configuration**

    Task definition name = my-ecs-frontend-task-definition

    **Infrastructure requirements**

    Launch type = Amazon EC2 Instance
  
    Operating System = Linux

    Network Module = bridge

    CPU = 1vCPU

    Memory = 1 Gb

    Task role = -

    Task Execution role = Create new role

    **Container**

    Container detail :

    Name = fc_frontend

    Image URL = Copy url image pada repository ECR

    Port Mappings :

    Host port = 80

    Container port = 80

    Protocol = TCP

    Portname = fc_frontend-80-tcp

    Matikan use log collective

    *biarkan settingan lainya default*

    "Create"

- Ulangi langkah tadi untuk membuat task definition untuk backend
- jangan lupa untuk mengganti nama, image url, dan port mappings

## 9. Membuat service di Cluster ECS

![Selection_016](https://github.com/user-attachments/assets/46c53c7a-e5ed-4ae8-a88f-1a7d5009c66b)

- Cari  ECS service di AWS Console

*Cara Pertama*
- Pilih cluster yang akan di buatkan Service
- Pilih Create pada kolom "Services"

    **Environtment**
  
    Biarkan default pada bagian compute configuration

    **Deployment configuration**

    Application type = Service

    Family = my-ecs-frontend-task-definition

    Version = 1(Latest)

    Service Name = my-ecs-frontend-svc

    Desired task = 1

    *pilih opsi load balancing*

    **Load balancing**

    Load balancing type = Application Load balancer

    Container = fc_frontend

    Loadbalancer = my-ecs-lb

    Listener = Use Existing listener

    Listener = 443HTTPS

    Target Group = Use existing target group

    Target group name = my-ecs-frontend-80

    "Create"

*Cara Kedua*

- Masuk ke bagian 'Task Definition'
- Pilih Task Definition Backend
- Masuk ke Rev 1
- Pilih 'Service' Pada dropdown 'Deploy'

  **Lakukan kembali langkah pada saat pembuatan service frontend**

  *Jangan lupa untuk mengganti attribut yang di butuhkan menjadi bacend*

- Cek kembali cluster yang sudah terbuat 2 service
- Jika Sudah up/Running kedua service coba akses website manggunakan domain yang sudah di daftarkan di Route53

## 10. Pembuatan cloudfront dan Integrasi ELB

![Selection_017](https://github.com/user-attachments/assets/11041bf3-da64-469a-9492-d43dca4f8765)

- Cari ACM pada AWS Console
- Cek dan pastikan sertifikat ACM terdapat pada regian global (US)
- Cari Cloudfront pada AWS Console
- Create Distribution

  **Origin**

  Origin domain = *(Copy-Paste domainyang terdapat pada loadbalancers)*

  Protocol = HTTP only

  *Biarakan default settingan yg lain*

  **Default cache behavior**

  Viewer protocol policy = Redirect HTTP to HTTPS

  Cache and origin request = Legacy cache setting

  *Biarakan default settingan yg lain*

  **Web application firewall (WAF)**

  Pilih "Do not enable security protections"

  **Setting**

  Costum SSL certificate = Pilih sertifikat global yang sudah terbuat

  *Biarkan default settingan yang lain*

  "Create Distribution"

- Ubah dan tambahkan rule pada loadbalancer HTTP(80)
- Coba akses halaman

## 11. Menyiapkan Record route53

![Selection_018](https://github.com/user-attachments/assets/9565cd1c-afb3-4e69-aa7f-97cc5c56a4d0)

- Cari Route53 pada AWS console
- Masuk ke DNS management
- Pilih domain yang sudah di pakai
- Pilih domain yang akan di arahkan ke cloudfront
- "Edit Records"

  Route traffic to = *Arahkan ke cloudfront yang lita buat*

  Choose Distribution = ecs-fastcampuss

  "Save"

- Coba test buka domain dari Route53
- Patikan domain mengarah ke Cloudfront

## 12. CI/CD Menggunakan AWS CodeSeries

![Selection_019](https://github.com/user-attachments/assets/a49e6e84-3d61-4cbf-a9f8-e93fb478aca8)

- Buka cloud9 Dan mauk ke directory frontend
- Buat file "buildspec.yml"

  ```
    version: 0.2
    
    phases:
      pre_build:
        commands:
          - echo Logging in to Amazon ECR...
          - aws --version
          - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin 111122223333.dkr.ecr.us-west-2.amazonaws.com
          - REPOSITORY_URI=012345678910.dkr.ecr.us-west-2.amazonaws.com/hello-world
          - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
          - IMAGE_TAG=${COMMIT_HASH:=latest}
      build:
        commands:
          - echo Build started on `date`
          - echo Building the Docker image...
          - docker build -t $REPOSITORY_URI:latest .
          - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
      post_build:
        commands:
          - echo Build completed on `date`
          - echo Pushing the Docker images...
          - docker push $REPOSITORY_URI:latest
          - docker push $REPOSITORY_URI:$IMAGE_TAG
          - echo Writing image definitions file...
          - printf '[{"name":"hello-world","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
    artifacts:
        files: imagedefinitions.json

  ```
- Ubah bagian 'aws ecr get-login-password' dengan link yang terdapat pada ECR
    
![Selection_008](https://github.com/user-attachments/assets/6bea35d2-c717-4507-89a8-b548748f0138)

- Ubah juga bagian 'REPOSITORY_URI=' menggunakan URI ECR
- Dan beri nama pada bagian " printf '[{"name":"hello-world","imageUri":"%s"}]' "
- Commit dan Push github
  ```
  
    git add .
    git commit -m "add buildspec"
    git push
    
  ```
### Setup codebuild
- Cari Codebuild pada AWS console
- "Cretae project"

  **Project configuration**

  Project name = frontend-codebuild

  **Source**

  Source 1 - primary

  Source provider = Github

  *jika belum terhubung silahkan hubungkan akun github dengan menggunakan token*

  Repository = Repository in my github account

  Github repository = (Link repo frontend)

  **Environtment**

  Environtment image = Managed image

  Operating sistem = Amazon Linux

  Runtime = Standard

  Image = default

  Image version = default

  Service role = New service role

  Role name = default

  *Drop Additional Configuration*

  Ceklis pada bagian "Privileged"

  **Buildspec**

  Build specifications = Use a buildspec file

  buildspec name = buildspec.yml

  *Biarkan default settingan lainya*

  "Create Build Project"

- Buat juga Project dan buildspec.yml untuk backend dengan langkah yang sama
- Jangan lupa untuk mengubah dan push git code pada buildspec.yml
- Memberikan akses untuk codebuild
- Cari IAM Pada AWS console
- Masuk ke bagian Roles
- Pilih codebuild-service-role (fronend/backend)
- "add permissions"
- Attach policies
- Cari dan pilih ECR
- "Add permissions"
- Coba start build yang sudah terbuat
- Cek apakah sudah terdaftar atau ter-build di halaman ECR

### Setup CodeDeploy

- Setup permission iam
- Cari IAM pada AWS console
- Masuk ke Bagian 'Roles'
- "Create role"

  **Trusted entity type**

  Pilih AWS service
  
  **Use case**

  Source of use case = CodeDeploy

  Specified = CodeDeploy ECS

  "Next"

  **Add permissions**

  *Biarkann default*

  "Next"

  **Role details**

  Role name = codedeploy-ecs-role

  Description = default
  
  "Create"

- Setup target group
- Cari Target Gruop di AWS console
- "Create target group"

  **Basic configuration**

  Choose Target group = Instance

  Target group name = my-ecs-frontend-build-80

  Protocol-Port = HTTP (80)

  IP address type = IPv4

  VPC = isw-vpc

  Protocol version = HTTP1

  **Health Checks**

  Health check protocol = HTTP

  Health check path = /

  "Next"

  "Create"

- Buat juga untuk backend dengan langkah yang sama
- Edit loadbalancers
- Masuk ke loadbalancers
- pilih "my-ecs-lb"
- "Add Listener"

  **LIstener Details**

  Protocol-Port = HTTPS (8443)

  Routing Actions = Forward to target groups

  Target group = my-ecs-frontend-deploy-80

  **Secure listener settings**

  Certtificate Source = From ACM

  Certificate = Pilih certificate yang sudah terbuat dahulu

  "Add"

- Membuat Deployment
- Masuk ke ECR dan pilih Service
- "Create"

  **Environtment**

  *Biarkan menggunakan settingan default*

  **Deploy Configuration**

  Application type = Service

  Family = My-ecs-frontend-tg

  Version = 1(Latest)

  Service name = my-ecs-frontend-deploy-svc

  Service type = Replica

  **Deployment Options**

  Deployment type = BlueGreeen deployment powered by CodeDeploy

  Dployment configuration = CodeDeployDefaultECS

  Service role for CodeDeploy = codedeploy-ecs-role

  **Load Balancing**

  Load balancing type = Application loadbalancers

  Container = fc_frontend 8080

  loadbalancers = my-ecs-lb

  ***Test Listener**

  pilih add a test listener

  "Use existing listener"

  Test listener = 8443 HTTPS

  ***Target Group***

  Target group 1 = Use existing target group

  Target group 1 name = my-ecs-frontend-80

  Target group 2 = Use existing target group

  Target group 2 name = my-ecs-frontend-deploy-8080

  "Create"

- Tunggu sampai sukses terbuat
- Check ke service CodeDeploy
- Pilih bagian Apps
- Jika sudah tersedia lanjut membuat Deployment
- Buat File baru bernama "appspec.yml"

  ```
  version: 0.0
    Resources:
      - TargetService:
          Type: AWS::ECS::Service
          Properties:
            TaskDefinition: "arn:aws:ecs:us-east-1:111222333444:task-definition/my-task-definition-family-name:1"
            LoadBalancerInfo:
              ContainerName: "SampleApplicationName"
              ContainerPort: 80
  ```

- Ubah pada bagian 'TaskDefinition' menggunakan TaskDefiniton yg sudah kita buat

![Selection_020](https://github.com/user-attachments/assets/e02a8790-1f06-47cc-a9fe-e8d16dee373c)

- Ubah juga nama container dengan "fc_frontend"
- Buat Bucket S3 untuk menyimpan file appspec.yml
- Cari S3 di AWS console
- "Create Bucket"

  **General configuration**

  AWS region = asia pasific (Jakarta)ap-southeast-3

  Bucket name = isw-appspec-bucket

  *Biarkan default settingan lainya*

  "Create"

- Cek apakah sudah terbuat
- Jika sudah terbuat "upload" file appspec.yml
- Coba jalankan distribusi CodeDeploy
- Cari Codedeploy pada console AWS
- Masuk ke "Applications'
- Pilih app "AppECS-my-ecs-cluster:my-ecs-deploy-svc"
- Mausk ke bagian deployment
- "Create deployments"

  **Deployment Settings**

  Applicatiopn = AppECS-my-ecs-cluster:my-ecs-deploy-svc

  Deployment group = *pilih deployment yg sudah tersedia*

  Revision Location = *copy-paste URI file appspec.yml dari S3*

  "Create Deployment"

- Tunggu hingga proses dployment selesai

### Setup CodePipeline

- Masuk ke directory file di Cloud9
- Buat file "TaskDef.json"
- Masukan informasi yang terdapat pada Task definition
  ```
      {
        "taskDefinitionArn": "arn:aws:ecs:ap-southeast-3:675327529402:task-definition/my-ecs-frontend-td:3",
        "containerDefinitions": [
            {
                "name": "fc_frontend",
                "image": "675327529402.dkr.ecr.ap-southeast-3.amazonaws.com/fc_frontend",
                "cpu": 0,
                "portMappings": [
                    {
                        "name": "fc_frontend-80-tcp",
                        "containerPort": 80,
                        "hostPort": 80,
                        "protocol": "tcp",
                        "appProtocol": "http"
                    }
                ],
                "essential": true,
                "environment": [],
                "environmentFiles": [],
                "mountPoints": [],
                "volumesFrom": [],
                "ulimits": [],
                "healthCheck": {
                    "command": [
                        "CMD-SHELL",
                        "curl -f http://localhost/ || exit 1"
                    ],
                    "interval": 5,
                    "timeout": 5,
                    "retries": 3
                },
                "systemControls": []
            }
        ],
        "family": "my-ecs-frontend-td",
        "executionRoleArn": "arn:aws:iam::675327529402:role/ecsTaskExecutionRole",
        "networkMode": "bridge",
        "revision": 3,
        "volumes": [],
        "status": "ACTIVE",
        "requiresAttributes": [
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.24"
            },
            {
                "name": "com.amazonaws.ecs.capability.ecr-auth"
            },
            {
                "name": "ecs.capability.container-health-check"
            },
            {
                "name": "ecs.capability.execution-role-ecr-pull"
            },
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
            }
        ],
        "placementConstraints": [],
        "compatibilities": [
            "EC2"
        ],
        "requiresCompatibilities": [
            "EC2"
        ],
        "cpu": "1024",
        "memory": "1024",
        "runtimePlatform": {
            "cpuArchitecture": "X86_64",
            "operatingSystemFamily": "LINUX"
        },
        "registeredAt": "2024-04-24T23:26:30.230Z",
        "registeredBy": "arn:aws:iam::675327529402:user/ibnu"
    }
  ```
- Ubah juga file "buildspec.yml"

  ```
  version: 0.2

      phases:
        pre_build:
          commands:
            - echo Logging in to Amazon ECR...
            - aws --version
            - aws ecr get-login-password --region ap-southeast-3 | docker login --username AWS --password-stdin 675327529402.dkr.ecr.ap-southeast-3.amazonaws.com
            - REPOSITORY_URI=675327529402.dkr.ecr.ap-southeast-3.amazonaws.com/fc_frontend
            - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
            - IMAGE_TAG=${COMMIT_HASH:=latest}
        build:
          commands:
            - echo Build started on `date`
            - echo Building the Docker image...
            - docker build -t $REPOSITORY_URI:latest .
            - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
        post_build:
          commands:
            - echo Build completed on `date`
            - echo Pushing the Docker images...
            - docker push $REPOSITORY_URI:latest
            - docker push $REPOSITORY_URI:$IMAGE_TAG
            - echo Writing image definitions file...
            - printf '[{"name":"fc_frontend","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      artifacts:
          files:
            - 'image*.json'
            - 'appspec.yaml'
            - 'taskdef.json'
          secondary-artifacts:
            DefinitionArtifact:
              files:
                - appspec.yaml
                - taskdef.json
            ImageArtifact:
              files:
                - imagedefinitions.json
   ```
- Commit and push github
  
   ```
    git add .
    git commit -m "add taskdef"
    git push
    
   ```

- Buat Pipeline di AWS
- Cari CodePipeline di Console AWS
- "Create Pipeline"

  **Pipeline Settings**

  Pipeline name = fc-frontend-pipeline

  *biarkan default settingan lainya*

  "Next"

  **Source**

  Source provider = Github (version-1)

  "Connect to github"

  Repository = fc_frontend

  Branch = main

  Change directors = AWS CodePipeline

  "Next"

  **Build**

  Build Provider = AWS CodeBuild

  Region = Asia pasific (Jakarta)

  Project name = fc-frontend-codebuild

  Build type = Single build

  "Next"

  **Deploy**

  Deploy provider = Amazon ECS Blue/green

  Region = Asia Pasific (Jakarta)

  AWS CodeDeploy Application name = AppECS-my-ecs-cluster:my-ecs-deploy-svc

  AWS CodeDeploy deployment group = *pilih deployment group yang tersedia*

  Amazon ECS Task definition = taskdef.json

  AWS CodeDeploy AppSpec file = appspec.yml

  "Next"

  "Create Pipeline"

- Tunggu hingga proses CI/CD selesai



  
