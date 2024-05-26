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




