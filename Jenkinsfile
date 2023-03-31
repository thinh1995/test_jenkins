// def BuildDokcerImage() {
//     sh 'docker build . -f ${DOCKER_FILE} -t ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}'
// }

// def PushDockerImage() {
//     sh 'echo $dockerhub_PSW | docker login -u $dockerhub_USR --password-stdin'
//     sh 'docker image tag ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_HUB}/${IMAGE_NAME}:${APP_ENV}'
//     sh 'docker push ${DOCKER_HUB}/${IMAGE_NAME}:${APP_ENV}'
// }

// def CleanUpDocker() {
//     sh 'docker rmi ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}'
//     sh 'docker image prune -f'
// }

pipeline {
    agent {
        label 'ssh-agent'
    }

    environment {
        APP_ENV = 'latest'
        IMAGE_NAME = 'test'
        DOCKER_HUB = 'thinh1995'
        DOCKER_FILE = 'Dockerfile'
        dockerhub = credentials('dockerhub')
        recipientEmails = "cuongthinhtuan2006@gmail.com"
    }

     stages {
        stage('Validate Pull Request') {
             when {
                changeRequest()
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github_account', passwordVariable: 'password', usernameVariable: 'username')]) {
                        pullRequest.setCredentials(username, password)
                    }

                    junit allowEmptyResults: true, skipPublishingChecks: true, testResults: 'test-results.xml'

                    echo "PR Number: ${pullRequest.number}"
                    echo "PR State ${pullRequest.state}"
                    echo "PR Target Branch ${pullRequest.base}"
                    echo "PR Source Branch ${pullRequest.headRef}"
                    echo "PR Can Merge ? ${pullRequest.mergeable}"

                    if (!pullRequest.mergeable) {
                        throw new Exception("PR has conflicting files!")
                    }

                    recordIssues tools: [java(), checkStyle(pattern: '**/build/**/main.xml', reportEncoding: 'UTF-8')]
                    recordCoverage(tools: [[parser: 'JACOCO']],
                        id: 'jacoco', name: 'JaCoCo Coverage',
                        sourceCodeRetention: 'EVERY_BUILD',
                        qualityGates: [
                                [threshold: 60.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                                [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]])
                
                    // sh "git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'"
                    // sh "git fetch --all"
                    // sh "git checkout origin/${pullRequest.base}"
                    // sh "git merge --no-edit origin/${pullRequest.headRef}"
                }
            }
        }

        // stage('Deploy Master') {
        //     when {
        //         branch 'master'
        //     }
        //     steps {
        //         BuildDokcerImage()
        //         PushDockerImage()
        //         CleanUpDocker()
        //     }
        // }
    }

    post {
        success {
            script {
                if (env.CHANGE_ID) {
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
                if (env.CHANGE_ID) {
                    pullRequest.createStatus(status: 'failure',
                                context: 'continuous-integration/jenkins/pr-merge/tests',
                                description: 'All tests are failed',
                                targetUrl: "${env.JOB_URL}/testResults")

                    pullRequest.labels = ['Build Failed']
                }
            }
        }
        always {
            script {
                if (env.BRANCH_NAME == 'master') {
                    mail to: "${recipientEmails}",
                    subject: "[Mollibox] Jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
                    body: "A new notification from Mollibox\n${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}\n\nJenkins,\nMollibox"
                }
            }

            cleanWs()
        }
    }
}