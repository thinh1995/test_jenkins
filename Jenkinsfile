def BuildDokcerImage() {
    sh 'docker build . -f ${DOCKER_FILE} -t ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}'
}

def PushDockerImage() {
    sh 'echo $dockerhub_PSW | docker login -u $dockerhub_USR --password-stdin'
    sh 'docker image tag ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_HUB}/${IMAGE_NAME}:${APP_ENV}'
    sh 'docker push ${DOCKER_HUB}/${IMAGE_NAME}:${APP_ENV}'
}

def CleanUpDocker() {
    sh 'docker rmi ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}'
    sh 'docker image prune -f'
}

pipeline {
    agent {
        label 'ssh-agent'
    }

    environment {
        dockerhub = credentials('dockerhub')
        APP_ENV = 'latest'
        IMAGE_NAME = 'test'
        DOCKER_HUB = 'thinh1995'
        DOCKER_FILE = 'Dockerfile'
        recipientEmails = "cuongthinhtuan2006@gmail.com"
    }

     stages {
        stage('Validate Pull Request') {
             when {
                changeRequest()
            }
            steps {
                script {
                    echo "PR ID: ${pullRequest.id}"
                    echo "PR State ${pullRequest.state}"
                    echo "PR Target branch ${pullRequest.base}"
                    echo "PR Source branch ${pullRequest.headRef}"
                    echo "PR can merge ${pullRequest.mergeable}"

                    if (!pullRequest.mergeable) {
                        throw new Exception("PR has conflicting files!")
                    }

                    sh "git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'"
                    sh "git fetch --all"
                    sh "git checkout origin/${pullRequest.base}"
                    sh "git merge --no-edit origin/${pullRequest.headRef}"

                    BuildDokcerImage()
                    CleanUpDocker()
                }
            }
        }

        stage('Deploy Master') {
            when {
                branch 'master'
            }
            steps {
                BuildDokcerImage()
                PushDockerImage()
                CleanUpDocker()
            }
        }
    }

    post {
        success {
            script {
                if (pullRequest.id) {
                    pullRequest.createStatus(status: 'success',
                                context: 'continuous-integration/jenkins/pr-merge/tests',
                                description: 'All tests are passing',
                                targetUrl: "${env.JOB_URL}/testResults")

                    pullRequest.labels = ['Build Success']
                }
            }
        }
        failure {
            script {
                if (pullRequest.id) {
                    pullRequest.createStatus(status: 'failure',
                                context: 'continuous-integration/jenkins/pr-merge/tests',
                                description: 'All tests are failed',
                                targetUrl: "${env.JOB_URL}/testResults")

                    pullRequest.labels = ['Build Failed']
                }
            }
        }
        always{
                mail to: "${recipientEmails}",
                subject: "[Mollibox] Jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
                body: "A new notification from Mollibox\n${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}\n\nJenkins,\nMollibox"

            cleanWs()
        }
    }
}