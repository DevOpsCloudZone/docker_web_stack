# Docker Push Stage – Jenkins Pipeline

## Docker Push Stage

### 1. `stage('Docker Push')`

```groovy
stage('Docker Push') {
```

This creates a Jenkins pipeline stage named **Docker Push**.

In Jenkins Stage View, the pipeline will look like:

```text
Docker Build
     ↓
Docker Push
```

The purpose of this stage is to take the Docker images built in the previous stage and upload them to Docker Hub.

---

### 2. `steps`

```groovy
steps {
```

The `steps` block contains the actual commands that Jenkins executes inside the `Docker Push` stage.

The basic structure is:

```text
stage
   ↓
steps
   ↓
commands
```

---

### 3. `withCredentials`

```groovy
withCredentials([
```

`withCredentials` is used to securely retrieve credentials stored in Jenkins and make them temporarily available to the commands inside the block.

Our Docker Hub credentials are stored in:

```text
Jenkins
   ↓
Credentials
   ↓
dockerhub_id
```

Jenkins retrieves these credentials only when this stage executes.

---

### 4. `usernamePassword`

```groovy
usernamePassword(
```

`usernamePassword` tells Jenkins what type of credential we are retrieving.

Our Jenkins credential contains:

```text
Username
Password / Access Token
```

Therefore, we use the `usernamePassword` credential binding.

---

### 5. `credentialsId`

```groovy
credentialsId: 'dockerhub_id',
```

`credentialsId` tells Jenkins which stored credential should be used.

Our credential ID is:

```text
dockerhub_id
```

The important distinction is:

```text
credentialsId
      ↓
dockerhub_id
      ↓
Jenkins Credentials
      ↓
Username + Password/Token
```

`dockerhub_id` is **not** the Docker Hub username.

It is the ID of the credential stored inside Jenkins.

---

### 6. `usernameVariable`

```groovy
usernameVariable: 'DOCKER_USERNAME',
```

This tells Jenkins to take the username stored in the Jenkins credential and temporarily expose it as an environment variable named:

```text
DOCKER_USERNAME
```

For example:

```text
Jenkins Credential
       ↓
Username = umesh2425
       ↓
DOCKER_USERNAME=umesh2425
```

The shell can then access it using:

```bash
$DOCKER_USERNAME
```

This avoids hard-coding the username directly into the authentication command.

---

### 7. `passwordVariable`

```groovy
passwordVariable: 'DOCKER_PASSWORD'
```

This tells Jenkins to take the password or Docker Hub access token from the stored credential and temporarily expose it as:

```text
DOCKER_PASSWORD
```

The flow is:

```text
Jenkins Credential
       ↓
Password / Token
       ↓
DOCKER_PASSWORD
```

We do **not** write the actual Docker Hub token inside the Jenkinsfile.

---

### 8. Complete credential configuration

The complete configuration is:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'dockerhub_id',
        usernameVariable: 'DOCKER_USERNAME',
        passwordVariable: 'DOCKER_PASSWORD'
    )
]) {
```

This means:

> Retrieve the Jenkins credential identified by `dockerhub_id` and temporarily expose its username as `DOCKER_USERNAME` and its password/token as `DOCKER_PASSWORD`.

---

### 9. Credential block `{ }`

```groovy
{
```

Everything inside this block has access to the temporary environment variables:

```text
DOCKER_USERNAME
DOCKER_PASSWORD
```

The structure is:

```text
withCredentials
      │
      ├── DOCKER_USERNAME
      ├── DOCKER_PASSWORD
      │
      └── Shell commands
```

---

### 10. `sh`

```groovy
sh '''
```

`sh` tells Jenkins to execute the following commands using the Linux shell on the Jenkins agent.

The triple single quotes allow us to execute multiple shell commands together:

```groovy
sh '''
    command1
    command2
    command3
'''
```

---

### 11. Docker Login

```bash
echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
```

This command authenticates the Jenkins server with Docker Hub.

The process is:

```text
DOCKER_PASSWORD
      ↓
    echo
      ↓
      |
      ↓
docker login
```

`$DOCKER_PASSWORD` retrieves the Docker Hub password/token temporarily provided by Jenkins.

`$DOCKER_USERNAME` retrieves the Docker Hub username.

The `-u` option specifies the Docker Hub username:

```bash
-u "$DOCKER_USERNAME"
```

The `--password-stdin` option tells Docker to read the password/token from standard input instead of asking for it interactively.

Therefore:

```bash
echo "$DOCKER_PASSWORD" |
docker login -u "$DOCKER_USERNAME" --password-stdin
```

is a safer approach than writing the password directly in the Jenkinsfile.

---

### 12. Push Application Image

```bash
docker push umesh2425/appstack:app
```

This uploads the application Docker image to Docker Hub.

The image was previously created during the Docker Build stage:

```bash
docker build -t umesh2425/appstack:app -f Docker-app/Dockerfile .
```

The local image is:

```text
umesh2425/appstack:app
```

The `docker push` command uploads it to:

```text
Docker Hub
└── umesh2425/appstack
    └── app
```

---

### 13. Push Database Image

```bash
docker push umesh2425/appstack:db
```

This uploads the database Docker image to Docker Hub.

The image was previously created using:

```bash
docker build -t umesh2425/appstack:db -f Docker-db/Dockerfile .
```

The image is:

```text
umesh2425/appstack:db
```

After the push, Docker Hub contains:

```text
umesh2425/appstack
├── app
└── db
```

---

### 14. Docker Logout

```bash
docker logout
```

After the images are pushed, Jenkins logs out from Docker Hub.

The overall authentication flow is:

```text
docker login
     ↓
Authenticated with Docker Hub
     ↓
Push application image
     ↓
Push database image
     ↓
docker logout
```

`docker logout` only removes the Docker Hub authentication information from the Docker client.

It does **not** delete the Docker images.

The images remain:

```text
Docker Hub
├── umesh2425/appstack:app
└── umesh2425/appstack:db
```

---

## 15. Complete Docker Push Stage

The complete Jenkins stage is:

```groovy
stage('Docker Push') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub_id',
                usernameVariable: 'DOCKER_USERNAME',
                passwordVariable: 'DOCKER_PASSWORD'
            )
        ]) {
            sh '''
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                docker push umesh2425/appstack:app
                docker push umesh2425/appstack:db
                docker logout
            '''
        }
    }
}
```

---

## 16. Complete Mental Picture

```text
                    Jenkins Server
                          │
                          ▼
                    Docker Build
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
          appstack:app       appstack:db
                 │                 │
                 └────────┬────────┘
                          ▼
                   withCredentials
                          │
                     dockerhub_id
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
              Username            Token
              umesh2425          ********
                 │                 │
                 └────────┬────────┘
                          ▼
                    docker login
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
             docker push       docker push
                 │                 │
                 ▼                 ▼
          appstack:app       appstack:db
                 │                 │
                 └────────┬────────┘
                          ▼
                     Docker Hub
                          │
                          ▼
                    docker logout
```

---

## 17. Important Names to Remember

There are three different concepts involved in authentication:

| Name | Meaning |
|---|---|
| `dockerhub_id` | Jenkins credential ID |
| `DOCKER_USERNAME` | Temporary environment variable |
| `umesh2425` | Actual Docker Hub username |

And the Docker image name:

```text
umesh2425/appstack:app
```

can be understood as:

```text
umesh2425 / appstack : app
     │          │       │
     │          │       └── Image tag
     │          └────────── Docker Hub repository
     └───────────────────── Docker Hub username
```

The database image uses:

```text
umesh2425/appstack:db
```

So the Docker Push stage takes the images created by the **Docker Build** stage, authenticates securely using Jenkins Credentials, pushes both images to Docker Hub, and finally logs out.
