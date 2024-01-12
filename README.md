# Project 20: CICD with Github Actions
In this project, we will build step by step a CICD pipeline using Github Actions. This project builds and deploys an App(Rhena App) with DevSecOps best practices in three steps: 
 - Runs necessary code tests(maven, checkstyle, junit, jacoco and quality gates);
 - Builds a docker image and stores in ECR; and finaly
 - Deploys the latest image to ECS. RDS is used to store the app user creds and details about the app.

## Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL

## Build the project
# Github setup
Login to your github account, open gitbash terminal on your local machine and clone the code
```git
  git clone https://github.com/Ndzenyuy/Project-20_Github-actions.git
```
Copy the files from Project-20_Github-actions.git to a new folder(CICD-with-GitActions) and initialise git for tracking
```git
  mkdir CICD-with-GitActions
  cp -r Project-20_Github-actions/* CICD-with-GitActions
  cd CICD-with-GitActions
  git init
  code .
```
Now VSCode opens, under source control, publish project to your github.

# Sonarcode analysis and quality gates
We need to start checking our workflow step by step, first we analyse our code for bugs with sonarqube
in the file CICD-with-GitActions/.github/workflows/main.yml replace all its contents with
```yml
name: github Actions
on: workflow_dispatch
  
jobs: 
  Testing:
    runs-on: ubuntu-latest
    steps:
      - name: Testing workflow
        uses: actions/checkout@v4

      - name: Maven test
        run: mvn test
      
      - name: Checkstyle
        run: mvn checkstyle:checkstyle
```
This portion of code does three things: downloads the sorcecode to github actions, runs maven test and runs mvn checkstyle. This is the first set of tests to be sure the code structure and styles are good for production and bug free. 
Now push this code and run it; -> 


# AWS IAM, ECR and RDS
# Docker build and publish to ECR
# ECS Setup
# Deploy

# Database
Here,we used Mysql DB 
MSQL DB Installation Steps for Linux ubuntu 14.04:
- $ sudo apt-get update
- $ sudo apt-get install mysql-server

Then look for the file :
- /src/main/resources/db_backup.sql
- db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < db_backup.sql
