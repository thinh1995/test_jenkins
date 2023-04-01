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
        stage('Unit Tests') {
            agent {
                docker {
                    image 'php:cli-alpine3.17'
                }
            }
            when {
                changeRequest()
            }
            steps {
                script {
                    junit allowEmptyResults: true, skipPublishingChecks: true, testResults: 'test-results.xml'

                    echo "PR Number: ${pullRequest.number}"
                    echo "PR State ${pullRequest.state}"
                    echo "PR Target Branch ${pullRequest.base}"
                    echo "PR Source Branch ${pullRequest.headRef}"
                    echo "PR Can Merge ? ${pullRequest.mergeable}"

                    if (!pullRequest.mergeable) {
                        throw new Exception("PR has conflicting files!")
                    }

                    // recordIssues tools: [php(pattern: '**/build/**/test-php.xml'),
                    //     phpCodeSniffer(pattern: '**/build/**/test-phpCodeSniffer.xml'),
                    //     phpStan(pattern: '**/build/**/test-phpStan.xml')],
                    //     aggregatingResults: 'true', id: 'php', name: 'PHP', filters: [includePackage('io.jenkins.plugins.analysis.*')]
                    // recordIssues tool: errorProne(), healthy: 1, unhealthy: 20
                    // recordIssues tools: [checkStyle(pattern: '**/build/**/test-checkStyle.xml', , reportEncoding: 'UTF-8'),
                    //     spotBugs(pattern: '**/build/**/test-spotBugs.xml'),
                    //     pmdParser(pattern: '**/build/**/test-pmdParser.xml'),
                    //     cpd(pattern: '**/build/**/test-cpd.xml')],
                    //     qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true]]
                    // publishCoverage adapters: [jacoco('**/*/jacoco.xml')], sourceFileResolver: sourceFiles('STORE_ALL_BUILD'), skipPublishingChecks: true
                
                    // sh "git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'"
                    // sh "git fetch --all"
                    // sh "git checkout origin/${pullRequest.base}"
                    // sh "git merge --no-edit origin/${pullRequest.headRef}"

                    sh 'php -v'
                    echo 'Installing Composer'
                    sh 'curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer'
                    echo 'Installing project composer dependencies...'
                    sh 'composer install'

                    sh 'vendor/bin/phpunit'
                    xunit([
                        thresholds: [
                            failed ( failureThreshold: "0" ),
                            skipped ( unstableThreshold: "0" )
                        ],
                        tools: [
                            PHPUnit(pattern: 'build/logs/junit.xml', stopProcessingIfError: true, failIfNotNew: true)
                        ]
                    ])
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'build/coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report (HTML)',
                        reportTitles: ''
                    ])
                    publishCoverage adapters: [coberturaAdapter('build/logs/cobertura.xml')]
                }
            }
        }

        stage('Static Analysis') {
            parallel {
                stage('CodeSniffer') {
                    steps {
                        sh 'vendor/bin/phpcs --standard=phpcs.xml .'
                    }
                }
                stage('PHP Compatibility Checks') {
                    steps {
                        sh 'vendor/bin/phpcs --standard=phpcs-compatibility.xml .'
                    }
                }
                stage('PHPStan') {
                    steps {
                        sh 'vendor/bin/phpstan analyse --error-format=checkstyle --no-progress -n . > build/logs/phpstan.checkstyle.xml'
                    }
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
        always {
            script {
                if (env.BRANCH_NAME == 'master') {
                    mail to: "${recipientEmails}",
                    subject: "[Mollibox] Jenkins build:${currentBuild.currentResult}: ${env.JOB_NAME}",
                    body: "A new notification from Mollibox\n${currentBuild.currentResult}: Job ${env.JOB_NAME}\nMore Info can be found here: ${env.BUILD_URL}\n\nJenkins,\nMollibox"
                }
            }

            recordIssues([
                sourceCodeEncoding: 'UTF-8',
                enabledForFailure: true,
                aggregatingResults: true,
                blameDisabled: true,
                referenceJobName: "repo-name/master",
                tools: [
                    phpCodeSniffer(id: 'phpcs', name: 'CodeSniffer', pattern: 'build/logs/phpcs.checkstyle.xml', reportEncoding: 'UTF-8'),
                    phpStan(id: 'phpstan', name: 'PHPStan', pattern: 'build/logs/phpstan.checkstyle.xml', reportEncoding: 'UTF-8'),
                    phpCodeSniffer(id: 'phpcompat', name: 'PHP Compatibility', pattern: 'build/logs/phpcs-compat.checkstyle.xml', reportEncoding: 'UTF-8')
                ]
            ])

            cleanWs()
        }
    }
}