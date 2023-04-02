def buildDokcerImage() {
    sh 'docker build . -f ${DOCKER_FILE} -t ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}'
}

def pushDockerImage() {
    sh 'echo $dockerhub_PSW | docker login -u $dockerhub_USR --password-stdin'
    sh 'docker image tag ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_HUB}/${IMAGE_NAME}:${APP_ENV}'
    sh 'docker push ${DOCKER_HUB}/${IMAGE_NAME}:${APP_ENV}'
}

def cleanUpDocker() {
    sh 'docker rmi ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}'
    sh 'docker image prune -f'
}

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
        stage('Get latest code') {
            when {
                changeRequest()
            }
            steps {
                script {
                    echo "PR Number: ${pullRequest.number}"
                    echo "PR State ${pullRequest.state}"
                    echo "PR Target Branch ${pullRequest.base}"
                    echo "PR Source Branch ${pullRequest.headRef}"
                    echo "PR Can Merge ? ${pullRequest.mergeable}"

                    if (!pullRequest.mergeable) {
                        throw new Exception("PR has conflicting files!")
                    }
                
                    // sh "cd ${WORKSPACE}"
                    sh "git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'"
                    sh "git fetch --all"
                    sh "git checkout origin/${pullRequest.base}"
                    sh "git merge --no-edit origin/${pullRequest.headRef}"
                }
            }
        }

        stage('Prepare dependencies') {
            agent {
                docker {
                    image 'sineverba/php8xc:latest'
                    args '-u root:sudo'
                    reuseNode true
                }
            }
            steps {
                echo 'Installing project composer dependencies...'
                sh 'composer install'
                sh 'cp .env.example .env'
            }
        }

        stage('Static Analysis') {
            parallel {
                stage('PHPUnit') {
                    agent {
                        docker {
                            image 'sineverba/php8xc:latest'
                            args '-u root:sudo'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'vendor/bin/phpunit'
                        sh "vendor/bin/phpunit --coverage-cobertura='build/logs/cobertura.xml' tests/"
                    }
                }
                stage('CodeSniffer') {
                    agent {
                        docker {
                            image 'sineverba/php8xc:latest'
                            args '-u root:sudo'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'vendor/bin/phpcs --standard=phpcs.xml'
                    }
                }
                stage('PHPStan') {
                    agent {
                        docker {
                            image 'sineverba/php8xc:latest'
                            args '-u root:sudo'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'vendor/bin/phpstan analyse --no-progress -c phpstan.neon > build/logs/phpstan.checkstyle.xml'
                    }
                }
            }
        }

        stage ('Buid Docker Image') {
            steps {
                buildDokcerImage()
            }
        }

        stage('Deploy Master') {
            when {
                branch 'master'
            }
            steps {
                pushDockerImage()
            }
        }
    }

    post {
        always {
            script {
                if (env.BRANCH_NAME == 'master') {
                    mail to: "${recipientEmails}",
                    subject: "[Mollibox] Jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
                    body: "A new notification from Mollibox\n${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}\n\nJenkins,\nMollibox"
                }

                withChecks('Integration Tests') {
                    junit allowEmptyResults: true, testResults: 'build/logs/*.xml'
                }

                publishCoverage adapters: [coberturaAdapter('build/logs/cobertura.xml')]

                recordIssues([
                    sourceCodeEncoding: 'UTF-8',
                    enabledForFailure: true,
                    aggregatingResults: true,
                    blameDisabled: true,
                    referenceJobName: "${env.JOB_NAME}",
                    tools: [
                        phpCodeSniffer(id: 'phpcs', name: 'CodeSniffer', pattern: 'build/logs/phpcs.checkstyle.xml', reportEncoding: 'UTF-8'),
                        phpStan(id: 'phpstan', name: 'PHPStan', pattern: 'build/logs/phpstan.checkstyle.xml', reportEncoding: 'UTF-8'),
                    ]
                ])

                def oldImageID = sh(script: "docker images -q  ${DOCKER_HUB}/${IMAGE_NAME}:${BUILD_NUMBER}", returnStdout: true)

                if ("${oldImageID}" != '' ) {
                    cleanUpDocker()
                }
            }

            cleanWs()
        }
    }
}