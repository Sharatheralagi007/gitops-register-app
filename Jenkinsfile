pipeline {
    agent { label "Jenkins-Agent" }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag to deploy (leave empty to use the newest tag on Docker Hub)')
    }

    environment {
        APP_NAME    = "register-app-pipeline"
        DOCKER_REPO = "sharatheralagi/register-app-pipeline"
    }

    stages {
        stage("Resolve Image Tag") {
            steps {
                script {
                    def tag = params.IMAGE_TAG?.trim()
                    if (!tag) {
                        tag = sh(returnStdout: true, script: '''
                            curl -sS -f -m 30 "https://hub.docker.com/v2/repositories/${DOCKER_REPO}/tags?page_size=100&ordering=last_updated" \
                            | python3 -c "import sys,json,re; r=json.load(sys.stdin)['results']; print(next(t['name'] for t in r if re.fullmatch(r'1\\.0\\.0-\\d+', t['name'])))"
                        ''').trim()
                    }
                    if (!tag) {
                        error("Could not determine an image tag.")
                    }
                    env.DEPLOY_TAG = tag
                    echo "Deploying tag: ${env.DEPLOY_TAG}"
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
                   sed -i 's|${APP_NAME}:.*|${APP_NAME}:${env.DEPLOY_TAG}|g' deployment.yaml
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
                       git diff --cached --quiet || git commit -m "Updated image tag to ${DEPLOY_TAG}"
                       git pull --rebase origin main
                       git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment manifest updated to ${env.DEPLOY_TAG}. Argo CD will sync it."
        }
        failure {
            echo "CD pipeline failed. Check the stage logs above."
        }
    }
}
