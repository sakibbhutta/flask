# Assignment#2
## Flask pushed on AWS
1.  Create a directory `/flask` with `Dockerfile` having content: 
```yml
FROM python:3.6
LABEL maintainer="lorenz.vanthillo@gmail.com"
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 8080
ENTRYPOINT ["python"]
CMD ["app/app.py"]
```
2.  Created a `requirements.txt` file containing required data in same directory with following content:
```console
flask==1.0.2
```
3.  create a file `app/app.py` with following content:
```console
from flask import Flask,render_template
import socket
import os

app = Flask(__name__)

color = os.environ.get('APP_COLOR')

@app.route("/")
def index():
    try:
        host_name = socket.gethostname()
        host_ip = socket.gethostbyname(host_name)
        return render_template('index.html', hostname=host_name, 
            ip=host_ip, color=color)
    except:
        return render_template('error.html')


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
```
 4. Create a file `app/templates/index.html` for the main index page with follwoing content:
```console
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Index page</title>
</head>
<body  style="background-color:{{ color }};">
The hostname of the container is <b>{{ hostname }}</b> and its IP is <b>{{ ip }}</b>.
</body>
</html>
```
5. Create a file `app/templates/error.html` for error page with follwoing content:
```console
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Index page</title>
</head>
<body  style="background-color:{{ color }};">
The hostname of the container is <b>{{ hostname }}</b> and its IP is <b>{{ ip }}</b>.
</body>
</html>
```
6. Run following command to build the image from `Dockerfile`:
```console
$ docker build -t flask -f Dockerfile .
```
7.  The following image has been built:
![image](https://user-images.githubusercontent.com/126319802/229425786-40876160-06c6-46b8-8980-5baba0aefc20.png)
8.  Run the container with `flask` image by using following command:
```console
$ docker run -e APP_COLOR=blue flask
```
9.  It will generate the following output:
![image](https://user-images.githubusercontent.com/126319802/229426210-11d94d9e-3d2f-4595-801f-72a603ef46d8.png)
11. Now, Create a repository `flask` in `github` and run following commands to push this flask application to github:
```console
echo "# flask" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/sakibbhutta/flask.git
git push -u origin main
```
12. Now, to build the image on `github` and send the built image directly to `AWS ECR`, we need to go to github Actions and create a 
`workflow file` which is named `docker-image.yml` with following content:
```console
name: Build and push image to AWS ECR

on:
  push:
    branches:
      - 'main'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
        name: Configure AWS credentials
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
          
        name: Login to Amazon ECR
        id: login-ecr
      - uses: aws-actions/amazon-ecr-login@v1
        
        name: Build, tag, and push image to Amazon ECR
        
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: flask_github
          IMAGE_TAG: flask
        
          run: | 
            docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
            docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```
13.  Now, go to `AWS Elastic Container Registory` and create a registory there with name `flask`
14.  Go to `Identity and Access Management (IAM)` option of `AWS` to create a neew user with following previlges: 
```console
AmazonEC2ContainerRegistryFullAccess
```
15. During user creation process, generate `Access key ID` and `secret key access` 
16. Now, To set the secret variables go to `settings` of `flask` repositry, go to `Secrets and Variables` and then `Action`. 
click on `New repository secret` and add  `Access key ID` and `secret key access` to AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY respectively: 
17.  Finally, click on `Start commit` option to commit the changes to `AWS-ECR`
18. Job ran successfully:
![image](https://user-images.githubusercontent.com/126319802/229429619-d3b48e5d-6a4c-449f-9094-a9eaf176b6c1.png)
19. To build the image and deploy the running application on `AWS Elastic Beanstalk` make following changes in `workflow file`
```console
name: Deploy master
on:
  push:
    branches:
    - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout source code
      uses: actions/checkout@v2

    - name: Generate deployment package
      run: zip -r deploy.zip . -x '*.git*'

    - name: Deploy to EB
      uses: einaregilsson/beanstalk-deploy@v21
      with:
        aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        application_name: flask
        environment_name: Flask-env-1
        version_label: 3.5.0
        region: us-east-1
        deployment_package: deploy.zip
```
20. The above changes make the application RUN on AWS and any changes/commits in github repository are
synchronized and are made on running running flask application on AWS.
21. For `AWS EB`, appliaction needs to be configured with 
```console
        Name of application: flask
        Environment name: Flask-env-1
        version: 3.5.0
```
