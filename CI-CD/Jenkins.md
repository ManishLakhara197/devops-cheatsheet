# Jenkins Cheatsheet

![](https://imgur.com/jWGs9lH.png)

**1. Introduction:**

- Jenkins is an open-source automation server that helps automate parts of software development related to building, testing, and deploying, facilitating continuous integration and delivery.

**2. Installation:**

- **Docker Installation:**

  ```bash
  docker run -d -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
  ```

- **Direct Installation:**

  - **For Ubuntu/Debian:**

    ```bash
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt install jenkins
    ```

  - **For CentOS/RHEL:**

    ```bash
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    sudo yum install jenkins
    ```

- **Access Jenkins:**
  - Visit `http://localhost:8080` in your web browser.

**3. Jenkins Pipeline:**

- **Declarative Pipeline:**

  ```groovy
  pipeline {
      agent any
      environment {
          MY_VAR = "value"
      }
      stages {
          stage('Checkout') {
              steps {
                  checkout scm
              }
          }
          stage('Build') {
              steps {
                  sh 'make'
              }
          }
          stage('Test') {
              steps {
                  sh 'make test'
              }
          }
          stage('Deploy') {
              steps {
                  sh 'make deploy'
              }
          }
      }
      post {
          success {
              echo 'Pipeline completed successfully!'
          }
          failure {
              echo 'Pipeline failed.'
          }
      }
  }
  ```

- **Scripted Pipeline:**

  ```groovy
  node {
      stage('Checkout') {
          checkout scm
      }
      stage('Build') {
          sh 'make'
      }
      stage('Test') {
          sh 'make test'
      }
      stage('Deploy') {
          sh 'make deploy'
      }
  }
  ```

**4. Common Jenkins Commands:**

- **Restart Jenkins:**

  ```bash
  sudo systemctl restart jenkins
  ```

- **Manage Jenkins from CLI:**

  ```bash
  java -jar jenkins-cli.jar -s http://localhost:8080/ list-jobs
  ```

**5. Useful Jenkins Plugins:**

- **Blue Ocean:** Modern UI for Jenkins pipelines.
- **Git:** Integrate Git version control into Jenkins.
- **Pipeline:** Enables Pipeline as Code.
- **Credentials Binding:** Securely manage credentials.
- **SonarQube Scanner:** Integrate code quality checks.
- **Slack Notification:** Send pipeline status notifications to Slack.

**6. Best Practices:**

- **Pipeline as Code:** Always use Jenkins Pipelines defined in `Jenkinsfile` for consistent and version-controlled builds.
- **Use Parameters:** Use parameters to make your pipelines flexible and reusable.

  ```groovy
  parameters {
      string(name: 'ENV', defaultValue: 'dev', description: 'Environment')
  }
  ```

- **Secure Jenkins:** Regularly update plugins, use RBAC, and secure the Jenkins instance with HTTPS.

**7. Jenkins Configuration:**

- **Manage Jenkins:**
  - Manage and configure global settings from the Jenkins dashboard under **Manage Jenkins**.
- **Configure Tools:** Set up JDK, Maven, and other tools globally in **Global Tool Configuration**.
- **Jenkinsfile Configuration:**
  - Define your pipeline stages, environment, and agents within a `Jenkinsfile` stored in your repository.

**8. Advanced Jenkins:**

- **Parallel Stages:**

  ```groovy
  pipeline {
      agent any
      stages {
          stage('Parallel') {
              parallel {
                  stage('Unit Tests') {
                      steps {
                          sh 'make test'
                      }
                  }
                  stage('Integration Tests') {
                      steps {
                          sh 'make integration-test'
                      }
                  }
              }
          }
      }
  }
  ```

- **Shared Libraries:** Centralize and reuse pipeline code across projects using Shared Libraries.

## **Troubleshooting**

### **Common Issues**

1. **Jenkins Won't Start**
   ```bash
   # Check logs
   sudo tail -f /var/log/jenkins/jenkins.log
   
   # Check permissions
   sudo chown -R jenkins:jenkins /var/lib/jenkins
   ```

2. **Pipeline Failure**
   ```groovy
   // Add error handling
   pipeline {
       agent any
       stages {
           stage('Build') {
               steps {
                   script {
                       try {
                           sh 'make build'
                       } catch (exc) {
                           echo 'Build failed!'
                           throw exc
                       }
                   }
               }
           }
       }
   }
   ```

3. **Plugin Issues**
   - Clear plugin cache:
     ```bash
     rm -rf $JENKINS_HOME/plugins/*.jpi
     rm -rf $JENKINS_HOME/plugins/*.hpi
     ```
   - Restart Jenkins after plugin updates

## **Useful Plugins**

1. **Pipeline**
   - Pipeline Graph View
   - Pipeline Stage View
   - Blue Ocean

2. **Source Control**
   - Git
   - GitHub Integration
   - BitBucket Integration

3. **Build Tools**
   - Maven Integration
   - Gradle
   - NodeJS

4. **Testing**
   - JUnit
   - Cobertura
   - SonarQube Scanner

5. **Deployment**
   - Docker
   - Kubernetes
   - AWS

## **Production level jenkins file**

```groovy
@Library('my-shared-library') _

pipeline {
    agent any

    parameters {
        choice(
            name: 'DEPLOY_ENV', 
            choices: ['staging', 'production'], 
            description: 'Target environment for GitOps update'
        )
        booleanParam(
            name: 'SKIP_TESTS', 
            defaultValue: false, 
            description: 'Skip unit testing and security scans if urgent'
        )
    }

    environment {
        APP_NAME         = 'ecom-frontend'
        IMAGE_NAME       = "my-docker-registry/${env.APP_NAME}"
        IMAGE_TAG        = "${env.BUILD_NUMBER}"
        
        // NEW GITOPS VARIABLES: Reference your infrastructure config repository
        GITOPS_REPO_URL  = 'https://github.com'
        GITOPS_CREDS_ID  = 'github-gitops-ssh-key' // Jenkins Credential ID for Git push
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        disableConcurrentBuilds()
        keepJenkinsSystemLog()
    }

    stages {
        stage('Initialize & Clean') {
            steps {
                echo "Starting GitOps Build #${env.BUILD_NUMBER}"
                cleanWs()
            }
        }

        stage('Install Dependencies') {
            steps {
                nodejs('NodeJS-20') { sh 'npm ci' }
            }
        }

        stage('Unit Tests') {
            when { expression { !params.SKIP_TESTS } }
            steps {
                nodejs('NodeJS-20') { sh 'npm test -- --coverage --watchAll=false' }
            }
        }

        stage('Security & Code Scan') {
            when { expression { !params.SKIP_TESTS } }
            steps {
                withSonarQubeEnv('SonarQube-Server') { sh 'npm run sonar-scanner' }
                timeout(time: 10, unit: 'MINUTES') { waitForQualityGate abortPipeline: true }
            }
        }

        stage('Build Artifact & Container') {
            steps {
                script {
                    nodejs('NodeJS-20') { sh 'npm run build' }
                    sh "docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Publish Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                }
            }
        }

        // NEW GITOPS STAGE: Replaces the 'Deploy Application' step completely
        stage('Update GitOps Repository') {
            steps {
                // 1. Clean the directory to create a fresh workspace for cloning the infra repo
                dir('gitops-workspace') {
                    // 2. Checkout your infrastructure repository using dedicated write credentials
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[url: env.GITOPS_REPO_URL, credentialsId: env.GITOPS_CREDS_ID]]
                    ])

                    script {
                        // 3. Update the specific image tag inside your infrastructure manifest file
                        // This example assumes a plain YAML structure under environments/env_name/
                        def manifestPath = "environments/${params.DEPLOY_ENV}/deployment.yaml"
                        
                        sh """
                            sed -i 's|image: ${env.IMAGE_NAME}:.*|image: ${env.IMAGE_NAME}:${env.IMAGE_TAG}|g' ${manifestPath}
                        """

                        // 4. Commit and push changes back to Git so Argo CD can see them
                        sh """
                            git config user.name "Jenkins CI"
                            git config user.email "jenkins-ci@yourcompany.com"
                            git add ${manifestPath}
                            git commit -m "chore(gitops): update ${env.APP_NAME} image to tag ${env.IMAGE_TAG} in ${params.DEPLOY_ENV} [skip ci]"
                            git push origin main
                        """
                    }
                }
            }
        }
    }

    post {
        success { notifySlack('#ops-alerts') }
        failure { notifySlack('#ops-alerts') }
    }
}

```
