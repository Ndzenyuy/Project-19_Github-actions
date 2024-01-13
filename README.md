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
## Project Architecture
![](architecture)

## Build the project
### Github setup
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
Now VSCode opens, under source control icon, publish project to your github.

# Sonarcode analysis and quality gates
We need to start checking our workflow step by step, first we analyse our code for bugs with sonarqube
in the file CICD-with-GitActions/.github/workflows/main.yml replace all its contents with
```yml
name: CICD-with-GitActions
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
This portion of code does three things: downloads the sourcecode to github actions, runs maven test and runs mvn checkstyle. This is the first set of tests to be sure the code structure and styles are good and bug free. 
Now push this code and run it; 
```git
git add .
git commit "testing workflow"
git push
```
Now to GitHub -> Actions -> CICD-with-GitActions -> run workflow -> branch name: main, run workflow. Now you'll have been sure that the workflow works smoothly
![](workflow-test)
![](workflow-test2)

### Sonarcloud setup
Create an account in [SonarCloud](https://www.sonarcloud.io) and link it to your github -> Create a new organization

- Create an organization manually
- Organization name: rhena (or give any random number after it if Rhena isn't valid) -> create organization
  ![Creating an organization]()
- Analyse new project in the next window
  ```
  Display name: app
  Project key: rhena_app -> next
  Previous version = true -> create project  
  ```
- Choose analysis method: Github actions
- Copy the sonar token to a sticky note
  ![Sonar token]()
- Create a quality gate
  
    Under organizations select Rhena -> Quality gates -> Create -> Name: actionsQG
    ![Creating quality gate]()
    Add conditions -> Where?: overall code -> Quality gate fails when: Bugs -> Operator: is greater than, Value 35
    Projects: rhena_app
  
- Secrets in Github
   In our project repo, under settings -> secrets and variables -> Actions -> new repository secret:
   add the following secrets(name: value):
    - SONAR_TOKEN : xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx (input the token retrieved from above when creating project)
    - SONAR_URL : sonarcloud.io
    - SONAR_ORGANIZATION: rhena
    - SONAR_PROJECT_KEY: rhena_app

 - Test the sonar scanner and the quality gates, replace the content of the workflow(main.yml) with the following
```yml
  name: github Actions
  on: [push, workflow_dispatch]
  env: 
    AWS_REGION: us-east-2
    ECR_REPOSITORY: rhenaapp
    ECS_SERVICE: rhenaapp-svc
    ECS_CLUSTER: rhena-appl
    ECS_TASK_DEFINITION: aws-files/taskdeffile.json
    CONTAINER_NAME: rhenaapp
    
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
  
  # Setup java 11 to be default (sonar-scanner requirement as of 5.x)
        - name: Set Java 11
          uses: actions/setup-java@v3
          with:
            distribution: 'temurin' # See 'Supported distributions' for available options
            java-version: '11'
  
        # Setup sonar-scanner
        - name: Setup SonarQube
          uses: warchant/setup-sonar-scanner@v7
  
        # Run sonar-scanner
        - name: SonarQube Scan
          run: sonar-scanner
            -Dsonar.host.url=${{ secrets.SONAR_URL }}
            -Dsonar.login=${{ secrets.SONAR_TOKEN }}
            -Dsonar.organization=${{ secrets.SONAR_ORGANIZATION }}
            -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
            -Dsonar.sources=src/
            -Dsonar.junit.reportsPath=target/surefire-reports/ 
            -Dsonar.jacoco.reportsPath=target/jacoco.exec 
            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
            -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/
  
                # Check the Quality Gate status.
        - name: SonarQube Quality Gate check
          id: sonarqube-quality-gate-check
          uses: sonarsource/sonarqube-quality-gate-action@master
        # Force to fail step after specific time.
          timeout-minutes: 5
          env:
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            SONAR_HOST_URL: ${{ secrets.SONAR_URL }} #OPTIONAL 
```
Now push to code to Github which will automatically trigger the workflow

 

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
