# VDT Midterm Assignments
## Table of Contents

- Contents
- I. 3 Tier Web Application Development
- II. Deploy Web Application Using DevOps Tools And Practices
  -  1. Containerization
  -  2. Continuous Integration
  -  3. Automation
  
## I. 3 Tier Web Application Development
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
- Create an app.py to implements the api
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

