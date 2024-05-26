# VDT Midterm Assignments
## Table of Contents

- Contents
- I. 3 Tier Web Application Development
- II. Deploy Web Application Using DevOps Tools And Practices
  -  1. Containerization
  -  2. Continuous Integration
  -  3. Automation
  
## I. 3 Tier Web Application Development
### Backend SourceCode Repo:
- Link: https://github.com/BaoICTHustK67/VDT_backend

### Frontend SourceCode Repo:
- Link: https://github.com/BaoICTHustK67/VDT_frontend

### Tech Stack
1. Front-end server: ReactJS
2. Back-end server:  Flask
3. Database server: PostgresSQL, sqlite (for testing purpose)
### Repository Structure
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/cfa2af60-8243-47bd-9320-8d91d0e896fe)

### Frontend UI/UX 
- I have create the FE using ReactJS library with data fetching from BE api
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/77ed641d-5850-4ec0-9b44-e5245287ca58)
### Backend Api
- The Api was developed by using Flask
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/f77769c1-23e6-4c3e-824b-aa4ed64f66dc)
- Required libraries in requirement.txt
```
flask
Flask-SQLAlchemy
flask-cors
psycopg2-binary
```
- Create an app.py to implements the api (CRUD)
```
from flask import request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask import Flask
from flask_cors import CORS
import os

app = Flask(__name__)
CORS(app)

app.config["SQLALCHEMY_DATABASE_URI"] = "postgresql://postgres:postgres@flask_db:5432/postgres"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)

class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=False, nullable=False)
    gender = db.Column(db.Integer, unique=False, nullable=False)
    school = db.Column(db.String(80), unique=False, nullable=False)

    def to_json(self):
        return {
            "id": self.id,
            "name": self.name,
            "gender": self.gender,
            "school": self.school,
        }
    
with app.app_context():    
    db.create_all()

@app.route('/')
def hello_world():
    return "Hello World"


@app.route("/students", methods=["GET"])
def get_students():
    students = Student.query.all()
    json_students = list(map(lambda x: x.to_json(), students))
    return jsonify({"students": json_students})

@app.route("/create_student", methods=["POST"])
def create_student():
    name = request.json.get("name")
    gender = request.json.get("gender")
    school = request.json.get("school")

    print(name, gender, school)

    if not (name or gender or school):
        return (
            jsonify({"message": "You must include a name, gender and school"}),
            400,
        )
    
    new_student = Student(name=name, gender=gender, school=school)
    try:
        db.session.add(new_student)
        db.session.commit()
    except Exception as e:
        return (
            jsonify({"message": str(e)}),
            400,
        )
    
    return jsonify({"message": "Student created!"}), 201

@app.route("/update_student/<int:student_id>", methods=["PATCH"])
def update_student(student_id):
    student = Student.query.get(student_id)

    if not student:
        return jsonify({"message": "Student not found!"}), 404
    
    data = request.json
    student.name = data.get("name", student.name)
    student.gender = data.get("gender", student.gender)
    student.school = data.get("school", student.school)

    db.session.commit()

    return jsonify({"message":"Student updated!"}), 200

@app.route("/delete_student/<int:student_id>", methods=["DELETE"])
def delete_student(student_id):
    student = Student.query.get(student_id)

    if not student:
        return jsonify({"message": "Student not found!"}), 404
    

    db.session.delete(student)
    db.session.commit()

    return jsonify({"message":"Student deleted!"}), 200
```
### Database using PostgresSQL
- I host an database on localhost with the following connection variables:
```
  POSTGRES_DB=flask_db
  POSTGRES_USER=postgres
  POSTGRES_PASSWORD=postgres
```
### Unit Tests for APIs
- I have write unit test for API using unnittest module
```
import unittest
import json
from backend.app import app, db
from app import Student

class TestAPI(unittest.TestCase):

    def setUp(self):
        app.config['TESTING'] = True
        app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///test.db'
        self.app = app.test_client()
        self.student_data = {
            "name": "Ba Bao",
            "gender": 0,
            "school": "HUST"
        }
        with app.app_context():
            db.create_all()

    def tearDown(self):
        with app.app_context():
            db.session.remove()
            db.drop_all()

    def test_create_student(self):
        response = self.app.post('/create_student', json=self.student_data)
        self.assertEqual(response.status_code, 201)

        with app.app_context():
            student = Student.query.filter_by(name=self.student_data["name"]).first()
            self.assertIsNotNone(student)

    def test_get_students(self):
        with app.app_context():
            new_student = Student(name="Ba Bao", gender=0, school="HUST")
            db.session.add(new_student)
            db.session.commit()

        response = self.app.get('/students')
        data = json.loads(response.data.decode('utf-8'))

        self.assertEqual(response.status_code, 200)
        self.assertTrue(len(data["students"]) > 0)

    def test_update_student(self):
        with app.app_context():
            new_student = Student(name="Ba Bao", gender=0, school="HUST")
            db.session.add(new_student)
            db.session.commit()

        update_data = {
            "name": "Updated Name",
            "gender": "Male"
        }

        response = self.app.patch('/update_student/1', json=update_data)
        self.assertEqual(response.status_code, 200)

        with app.app_context():
            updated_student = Student.query.filter_by(id=1).first()
            self.assertEqual(updated_student.name, update_data["name"])
            self.assertEqual(updated_student.gender, update_data["gender"])

    def test_delete_student(self):
        with app.app_context():
            new_student = Student(name="Ba Bao", gender=0, school="HUST")
            db.session.add(new_student)
            db.session.commit()

        response = self.app.delete('/delete_student/1')
        self.assertEqual(response.status_code, 200)

        # Check if the student is actually deleted from the database
        with app.app_context():
            deleted_student = Student.query.filter_by(id=1).first()
            self.assertIsNone(deleted_student)

if __name__ == '__main__':
    unittest.main()

```
- Testing Result
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/b96f1147-9eca-41db-9013-5149fd304262)

## II. Deploy Web Application Using DevOps Tools And Practices
### 1. Containerization
- Dockerfile for Frontend with ReactJS
```
# Use Node.js 20 image as base
FROM node:20-alpine

# Set working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json .

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .
```
- Build Result for Frontend Image
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/04e6eeb8-b29c-49d8-a87f-9d6d97d0bc62)

-Build History for Frontend Image
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/139499d8-8161-4064-9985-2a7fca724ed6)

- Dockerfile for Backend with Flask
```
# Use the official Python image as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

COPY requirements.txt ./

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 4000

CMD [ "flask", "run", "--host=0.0.0.0", "--port=4000"]

```
- Build Result for Backend Image
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/6777c735-eacf-425f-97c5-6a38081554ca)

- Build History for Backend Image
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/3ff81daa-b2f5-4137-98d6-d9bc17a2ef11)

### 2.Continuous Integration
- I use Github Actions as the CI tool
- Create an python-app.yaml file inside the .github/workflows folders

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/b32fe7ce-cb9b-49b8-a342-96216ce02ce0)

- Content of the python-app.yaml file
```
name: CI

on:
  push:
    branches:
      - main  
  pull_request:
    branches:
      - main  

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.11'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run tests
      env:
        FLASK_ENV: test  
      run: |
        python test_api.py
```
- Output log of the CI workflow
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/c507ce63-0d69-4d77-9c60-ecc1691beaac)
  - Set up job
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/c6daa2e9-79cf-43be-8506-7f434b5cd80e)

  - Checkout code
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/f8e574fb-3ddd-4462-ac54-089068fa1da3)

  - Set up Python
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/40bdc40b-7ab1-4163-91d7-5fd85e5a0407)

  - Install dependencies
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/81b8ad2b-a441-4672-ae07-cbc5d506c3bd)
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/8828bae7-1fe6-47b8-94e3-79a9109aef37)


  - Run tests
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/a338e619-6412-4604-98da-1c80a7a554d0)

  
  - Post Set up Python
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/8ba12521-051a-44c8-ac3e-3ebab02eda48)

  - Post Checkout code
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/9b97b37a-aec4-4968-8de0-5d3f40ea53fb)

  - Complete job     
  ![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/1d7009e4-4bff-4a23-8229-0e8dafad6dfb)
### 3. Automation 
- SourceCode of the ansible playbook: https://github.com/BaoICTHustK67/VDT_ansible_playbook
- Ansible playbook with one role per service

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/1638bd66-830a-4f17-921c-0da2c2df6972)

- Service Configurations through variables
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/94824171-a068-4f6b-92ea-9510093c6032)

- Multihost deployment (each host per service)

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/ae1c39c4-c386-4068-bd81-3a2b1fff005a)

#### Step by step of docker images deployment
1. Create 3 instances of VM with configurations that allow ssh between them and my local machine

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/9c43a737-858b-4279-b37c-fba074b0ba1b)

2. Install ansible on the local machine (Ubuntu)
```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansibl
```
3. Building the ansible-playbook folder with correct configuration with your ec2 context
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/6fb53cfb-e9eb-4539-9d7b-33fda8fec022)

4. Running the ansible playbook
```
 ansible-playbook -i hosts deploy_playbook.yaml
```
5. Ansible Logging Result
- Database Service

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/ed9dc7cb-d1cd-4cae-8541-28f9defc1862)

- Backend Service

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/29b21ac6-7e6c-4e05-9d7d-59b21abefbcc)

- Web Service
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/58c6c41c-81d8-48cb-939f-0f798aed14c8)

6. Host Deploying Result
- Database Service
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/a3117b56-8410-4411-a967-fd7677d41fe5)


- Backend Service
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/951f83d7-25ec-42b1-be62-7408cb0ade5a)



- Web Service
  
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/79c3a015-2321-4543-99b0-3781129380bd)

# III. Research Report
## Title: Apache Nessie Git-like Solution for Data Version Control on Kubernetes
## Contents
## 1. Ý tưởng

## 1. Ý tưởng
Với lượng data ngày càng lớn, đa dạng ở nhiều format(hình ảnh, âm thanh, text, ...), đến từ nhiều nguồn khác nhau, việc kiểm soát và quản lý data luôn là vấn đề nhức nhối đặc biệt là ở các lĩnh vực như là Big Data, ML, AI,... nơi mà luôn cần nhiều dữ liệu thực tế. Vì vậy rất là cần thiết để có thể phát triển một giải pháp mà các nhà khoa học, nhà phát triển dữ liệu và những người khác có thể sử dụng để tạo thử nghiệm của họ một cách dễ dàng và thành công. Để kiểm tra và sửa đổi dữ liệu một cách an toàn, em đã chọn ra một số ứng dụng khá ổn định về mặt hiệu năng và giá cả

Người dùng có thể gửi Spark "submits" tới cụm K8s sử dụng Jupyter Notebook, và cụm K8s sẽ xử lý các khối lượng công việc này và lưu trữ dữ liệu kết quả trên một nhánh cụ thể của Iceberg (quản lý lớp metadata của dữ liệu) do Nessie catalog quản lý

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/a779af4c-879a-4039-b0da-0021d4d817a1)


## 2. Một số thuật ngữ cần phải biết
### Metastore là gì?
Metastore cung cấp kho lưu trữ siêu dữ liệu trung tâm có thể dễ dàng phân tích để đưa ra các quyết định tối ưu dựa trên dữ liệu và do đó, nó là một thành phần quan trọng của nhiều kiến ​​trúc data lake

### Nessie là gì?
Nessie là một intelligent metastore dành cho Apache Iceberg. Nó cung cấp một giải pháp thay thế hiện đại của Hive Metastore cho Iceberg tables và views , đồng thời cung cấp nhiều tính năng nâng cao để có các kiến trúc data lake hiệu quả hơn. Những tính năng này bao gồm:
  - Thêm hoặc thay đổi dữ liệu trên một nhánh, kiểm tra chất lượng của nhánh đó và hợp nhất các thay đổi đối với tính khả dụng chung của người dùng, tất cả trong cùng một data lake và không ảnh hưởng đến dữ liệu trong môi trường production
  -  Tạo các phiên bản dữ liệu chuyên biệt cho các trường hợp sử dụng cụ thể
  -  Cập nhật nhất quán nhiều bảng, với nhiều thay đổi, do đó loại bỏ sự thiếu nhất quán và sai lệch dữ liệu ở giữa chuỗi thay đổi

### Lợi ích của Nessie 
- So sánh giữa Nessie Metastore và Hive Metastore
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/62b9ac40-f87a-4eb3-8e89-a01794e2f690)
- Nessie hỗ trợ nhiều cơ sở dữ liệu RDBMS và NOSQL:

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/35956c9a-067f-4e41-8680-e9faf340cc9d)

- Với Nessie/Iceberg, bạn có thể thực hiện các thao tác giống như git trong bảng Iceberg, chẳng hạn như tạo nhánh, hợp nhất dữ liệu, v.v. Nó mang lại cho các nhà khoa học dữ liệu rất nhiều sự linh hoạt vì họ có thể thực hiện một bản sao riêng biệt của dữ liệu
- Thay cho giao thức hive (lỗi thời) JDBC và trifft, nó hỗ trợ giao thức stardadize HTTP thông qua (rest api)

### Minio là gì?
- MinIO là một giải pháp lưu trữ đối tượng (object storage) cung cấp API tương thích với Amazon Web Services S3 và hỗ trợ tất cả các tính năng cốt lõi của S3

### Spark là gì? - Liệu có phải là một compute engine?
- Apache Spark là một công cụ phân tích hợp nhất để xử lý dữ liệu quy mô lớn. Nó cung cấp các API cấp cao trong Java, Scala, Python và R và một công cụ được tối ưu hóa hỗ trợ các biểu đồ thực thi chung. Nó cũng hỗ trợ một bộ công cụ cấp cao hơn bao gồm Spark SQL cho SQL và xử lý dữ liệu có cấu trúc, pandas API trên Spark để xử lý công việc của pandas, MLlib cho máy học, GraphX ​​để xử lý đồ thị và Structured Streaming để tính toán gia tăng và xử lý luồng

### Cách hướng tiếp cận khi dùng Spark?
- Client Mode:
  Khi người dùng gửi một công việc từ pyspark hoặc spark contexts, spark context sẽ gửi một api đến cụm K8s để tạo các pods trong namespace được chỉ định. Các nhóm này sẽ xử lý tác vụ đồng thời và sau khi hoàn thành, nó sẽ quay trở lại spark driver thông qua headless service => chạy spark và tạo pods trên một cụm đơn lẻ => có thể quá tải tài nguyên
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/ace4de41-f965-402e-8317-ba174df258af)

- Cluster Mode:
  Ở cluster mode, mọi thứ hoạt động gần như chính xác như ở client mode, nhưng K8s chịu trách nhiệm tạo driver và executors. K8s sẽ tạo ra tất cả các spark infrastructure (driver và executors) để xử lý các job và spark-context sẽ kiểm soát và verify job nhận được từ driver => có thể chạy spark và tạo pods đa cụm tránh việc bị quá tài nguyên (auto-scaling,...)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/7ac4c331-8dad-4c90-8a39-a88b97c712e6)

### Jupyter Notebook - Hub là gì?
Jupyter Notebook là ứng dụng web để tạo và chia sẻ tài liệu code


## Triển khai giải pháp

### Một số điều kiện triển khai giải pháp
- Một cụm K8s
- Sử dụng helm để cài đặt các dịch vụ

### Cài đặt Nessie Database (MongoDB)
Chúng ta có thể bắt đầu triển khai cơ sở dữ liệu Nessie sau khi K8s hoạt động và đã cài đặt Helm. Chúng ta phải tạo PV K8s để duy trì dữ liệu trong một thư mục cụ thể
- Đối với mỗi pod/replica mongodb, tạo ba thư mục để lưu trữ dữ liệu lâu dài:
```
mkdir -p /data/volumes/mongodb-0 /data/volumes/mongodb-1 /data/volumes/mongodb-2
```
- Tạo một namespace riêng để tiện cho việc quản lý và bảo mật, đặt tên là nessie-database
```
kubectl create namespace nessie-database
kubectl config set-context --current --namespace=nessie-database
```
- Tạo một local storage class:
```
cat << EOF | kubectl apply -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
```
```
kubectl patch storageclass local-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
- Tạo PersistentVolume với storage class đã tạo và PersistentVolume sẽ nằm ở worker node được chỉ định (bao-workernode)
```
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: datadir-mongodb-0
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /data/volumes/mongodb-0
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - bao-workernode
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: datadir-mongodb-1
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /data/volumes/mongodb-1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - bao-workernode
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: datadir-mongodb-2
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /data/volumes/mongodb-2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - bao-workernode
EOF
```
- Kiểm tra rằng PV đã được tạo thành công và sẵn sàng (trạng thái Available)
```
kubectl get pv
```
- Sử dụng helm để deploy MongoDb với thông tin setup của nessie
```
helm install --namespace nessie-database  mongodb oci://registry-1.docker.io/bitnamicharts/mongodb  \
--set replicaCount=3  --set architecture=replicaset  --set auth.databases[0]="nessie" \
--set auth.replicaSetKey=myreplicasetkey  --set auth.usernames[0]="nessie" \
--set auth.passwords[0]="nessiepassword"   --set auth.rootPassword=rootpassword \
--set persistence.storageClass=local-storage
```
- Kiểm tra xem helm đã được deploy thành công chưa nếu rồi kiểu tra xem các pv đã nhận được pvc và mount vào các pod hay chưa
```
helm list
kubectl get pv
```
### Cài đặt Nessie
- Tạo namespace tương ứng với ứng dụng
```
kubectl create ns nessie
```
- Chuyển context k8s sang namespace của nessie và tạo một file mongodb-creds chứa thông tin kết nối đến mongodb => sau đó tạo một k8s secret tương ứng với file đã tạo
```
kubectl create secret generic mongodb-creds --from-env-file="$PWD/mongodb-creds"
```
- Cải đặt mongodb sử dụng helm chart với connection string đã được setup ở trên
```
helm repo add nessie-helm https://charts.projectnessie.org
helm install --namespace nessie nessie nessie-helm/nessie --set versionStoreType=MONGODB --set mongodb.connectionString="mongodb://mongodb-0.mongodb-headless.nessie-database.svc.cluster.local:27017\,mongodb://mongodb-1.mongodb-headless.nessie-database.svc.cluster.local:27017\,mongodb://mongodb-2.mongodb-headless.nessie-database.svc.cluster.local:27017"
```
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/616a3a23-cd09-46c4-9b0a-5bb972b3cc0a)

- Kiểm tra xem cài đặt đã chạy ổn định hay chưa qua các câu lệnh
```
helm list
kubectl get deploy
kubectl get po
```
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/a4836313-c34e-479d-95ee-c1fca8676588)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/9dad82f6-2e6a-4930-a1cd-8e5d694c4848)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/05e0b755-5928-45fc-8a35-aa0342ace2b1)

- Kiểm tra logs của pods để xem kết nối đến MongoDB

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/4f6a0c74-533a-44b7-b718-68691f5421e7)

- Kiểm tra service Nessie đã chạy ổn định hay chưa ở port 19120 sử dụng curl
```
kubectl get svc

curl <svc cluster address>:<Port>
```

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/110a65c4-7763-4bf8-bb0c-156086402764)

- Sử dụng port-forward để truy cập UI
```
nohup kubectl --namespace nessie port-forward svc/nessie 19120:19120 --address='192.168.137.10' &
```

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/98834aa0-755d-4e9c-a543-ec2570b821c1)

- Kiểm tra xem thông tin data ở các branch của nessie liệu đã khớp với dữ liệu ở cụm mongodb chưa
```
kubectl config set-context --current --namespace=nessie-database
Kubectl exec -it mongodb-0 -- mongosh --quiet -u nessie -p nessiepassword --authenticationDatabase nessie

use nessie
show dbs
show collections
db-refs.find
```

### Cài đặt Minio
- Cách cài đặt của Minio khá đơn giản, đầu tiên chúng ta sẽ tạo pv ở ngoài node và mount vào pod để lưu trữ giữ liệu lâu dài (phòng trường hợp node chết)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/e339cec1-6084-4d89-9ee9-39ac21250d64)

- Cài đặt Minio thông qua helm chart có sẵn với các tham số để namespace, user, storage, ...
```
helm install minio -n minio --create-namespace --set auth.rootUser=minio-admin --set auth.rootPassword=minio-secret-password --set persistence.storageClass=local-storage oci://registry-1.docker.io/bitnamicharts/minio
```

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/c89ee529-b14c-4ab7-8d80-30f3fb1036ac)

- Kiểm tra xem pvc đã "bound" và deployment đã "ready" hay chưa
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/7d795b64-42d6-4025-8cf4-483e03834927)

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/f912e404-022c-4dca-8731-1d2538ff7b54)

- Sử dụng port-forward để có thể tương tác với UI của Minio
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/6867b7f0-5add-4975-a874-a42ff9bda87c)

![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/e42f4870-bde4-42ff-a656-9ee15b1adb67)

- Tạo một bucket với tên bất kì (trong trường hợp này là warehouse)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/39b82a41-1a13-4a8d-8949-20aab8021336)

- Tạo và save thông tin access key để có thể sử dụng khi tạo spark context
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/58b86c98-aadb-492e-a903-6845c55c104c)


### Cài đặt Spark
- Tạo một namespace spark trên k8s để tiện cho việc quản lý
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/9833639f-7e31-41d7-8bff-5bdf8d1b4697)

- Tạo service account để cho phép spark account có thể truy cập vào Kubernetes API server để tạo và xem những pods thực thi
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/d7326b03-ddcf-41f4-a1e4-8973655d23ba)

- Tạo một clusterrolebinding gọi là spark-role liên kết serviceaccount nhằm chỉnh sửa perm trên namespace spark
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/461e783f-e646-4f4c-8c8e-bfcc9feb95b1)

- Tạo spark image của các executor để triển khai
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/025d9c41-7dd6-43f0-bade-520b3729b60a)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/70857de9-9410-45be-a4e7-4410c88eaf89)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/0f7cbf05-2c92-454f-90b0-92ba68266748)
![image](https://github.com/BaoICTHustK67/HoangBaBao/assets/123657319/1782c8e3-6eba-4b45-8fa2-8cb2b78024d5)


### Cài đặt Jupyter Notebook












