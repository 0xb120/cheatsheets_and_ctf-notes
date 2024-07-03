>[!question] What is Jenkins?
>Jenkins is _a self-contained Java-based program_, ready to run out-of-the-box, with packages for Windows, Linux, macOS and other Unix-like operating systems.
>The leading open source automation server, Jenkins provides hundreds of plugins to support building, deploying and automating any project.

## Jenkins pipeline & Jenkinsfile

*Jenkins Pipeline* is a suite of plugins which supports implementing and integrating continuous delivery pipelines into Jenkins. A continuous delivery (CD) pipeline is an automated expression of your process for getting software from version control right through to your users and customers. 

The definition of a Jenkins Pipeline is written into a text file (called a **Jenkinsfile** [^jenkinsfile]) which in turn can be committed to a project’s source control repository. This is the foundation of “Pipeline-as-code”; treating the CD pipeline as a part of the application to be versioned and reviewed like any other code. [^pipeline]

[^pipeline]: [Pipeline](https://www.jenkins.io/doc/book/pipeline/), jenkins.io
[^jenkinsfile]: [Using a Jenkinsfile](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/), jenkins.io

*Jenkinsfile*:
```json
pipeline {
    agent any
    environment {
        PROJECT = "src/urllib3"
    }

    stages {
        stage ('Install_Requirements') {
            steps {
                sh """
                    virtualenv venv
                    pip3 install -r requirements.txt || true
                """
            }
        }

        stage ('Lint') {
            steps {
                sh "pylint ${PROJECT} || true"
            }
        }

        stage ('Unit Tests') {
            steps {
                sh "pytest"
            }
        }

    }
    post {
        always {
            cleanWs()
        }
    }
}
```

Jenkins Pipeline exposes environment variables via the global variable `env`, which is available from anywhere within a `Jenkinsfile`. The full list of environment variables accessible from within Jenkins Pipeline is documented at `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env`
## Secrets and credentials

Jenkins' declarative Pipeline syntax [^pipeline-syntax] has the `credentials()` helper method (used within the [environment](https://www.jenkins.io/doc/book/pipeline/syntax/#environment) directive) which supports secret text, username and password, as well as secret file credentials.

[^pipeline-syntax]: [Pipeline Syntaxt](https://www.jenkins.io/doc/book/pipeline/syntax/), jenkins.io

The following Pipeline code shows an example of how to create a Pipeline using environment variables for secret text credentials. [^creds]

[^creds]: [Handling credentials](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#handling-credentials), jenkins.io

```json
pipeline {
    agent {
        // Define agent details here
    }
    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }
    stages {
        stage('Example stage 1') {
            steps {
                sh "echo $AWS_ACCESS_KEY_ID"
            }
        }
        stage('Example stage 2') {
            steps {
                // 
            }
        }
    }
}
```