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


