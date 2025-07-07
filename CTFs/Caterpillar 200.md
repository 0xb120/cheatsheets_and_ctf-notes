---
Category:
  - CI/CD
Difficulty: Easy
Platform: CICD Goat
tags: 
Status: 1. In progress
---
>[!quote]
Who. Are. You? You just have read permissions… is that enough? Use your access to the *Wonderland/caterpillar* repository to steal the flag2 secret, which is stored in the Jenkins credential store [^jenkins-creds].

[^jenkins-creds]: [Secrets and credentials in Jenkins](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Jenkins.md#Secrets%20and%20credentials)

Scenario:
- Jenkinsfile [^jenkinsfile] is protected: the pipeline is configured in the same repository from where the application code is stored at, but the current user can’t change it
- There are two jobs in Jenkins: *-prod* and *-test*.
- Think about this repository like an “open source” one… And escalate your privileges to control it.

[^jenkinsfile]: [Jenkins pipeline & Jenkinsfile](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Jenkins.md#Jenkins%20pipeline%20&%20Jenkinsfile)

Hint:
- Fork the repository and create a pull request to trigger a malicious pipeline.
- After you execute your malicious code in the Jenkins job, what environment variables can help you move forward?
- Found that Gitea access token from the pipeline? Great. There’s another pipeline which is triggered by pushing to the main branch. Maybe you can access the flag from there!

Solution:
- [CICD-SEC-4 Public Poisoned Pipeline Execution (3PE)](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/CICD%20security.md#CICD-SEC-4%20Poisoned%20Pipeline%20Execution)
- https://github.com/cider-security-research/cicd-goat/blob/main/solutions/caterpillar.md
	- You can both: remove the condition for the attack and do it via PR or leave it and push directly on main branch instead. 

---


Cloned and inspected the repository:
```bash
$ git clone http://127.0.0.1:3000/Wonderland/caterpillar
$ ls
CHANGELOG.rst  CONTRIBUTING.rst  docs  Jenkinsfile  LICENSE  loguru  MANIFEST.in  README.rst  setup.py  tests  tox.ini
$ cat Jenkinsfile
pipeline {
    agent any
    environment {
        PROJECT = "loguru"
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
                sh "pytest || true"
            }
        }

        stage('deploy') {
            when {
                expression {
                    env.BRANCH_NAME == 'main'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'flag2', usernameVariable: 'flag2', passwordVariable: 'TOKEN')]) {
                    sh 'curl -isSL "http://wonderland:1234/api/user" -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" || true'
                }
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