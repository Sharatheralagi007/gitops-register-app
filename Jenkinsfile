pipeline {
    agent { label "Jenkins-Agent" }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag to deploy')
    }

    environment {
        APP_NAME = "register-app-pipeline"
    }

    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (!params.IMAGE_TAG?.trim()) {
                        error("IMAGE_TAG is empty. Trigger this job with IMAGE_TAG=<tag>.")
                    }
                }
            }
        }

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/Sharatheralagi007/gitops-register-app.git'
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                   echo "--- Before ---"
                   cat deployment.yaml
                   sed -i 's|${APP_NAME}:.*|${APP_NAME}:${params.IMAGE_TAG}|g' deployment.yaml
                   echo "--- After ---"
                   cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh '''
                       git config user.name "sharatheralagi007"
                       git config user.email "sharatheralagi@gmail.com"
                       git add deployment.yaml
                       git diff --cached --quiet || git commit -m "Updated image tag to ${IMAGE_TAG}"
                       git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment manifest updated to ${params.IMAGE_TAG}. Argo CD will sync it."
        }
        failure {
            echo "CD pipeline failed. Check the stage logs above."
        }
    }
}
