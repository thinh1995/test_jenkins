pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Validate Pull Request') {
             when {
                changeRequest()
            }
            steps {
                script {
                    echo "Current PR ID: ${pullRequest.id}"
                    echo "Current PR State ${pullRequest.state}"
                    echo "PR Target branch ${pullRequest.base}"
                    echo "PR Source branch ${pullRequest.headRef}"
                    echo "PR can merge ${pullRequest.mergeable}"

                    if (!pullRequest.mergeable) {
                        throw new Exception("PR has conflicts!")
                    }

                    sh "git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'"
                    sh "git fetch --all"
                    sh "git checkout origin/${pullRequest.base}"
                    sh "git merge --no-edit origin/${pullRequest.headRef}"
                }
            }
        }

        stage('Deploy Master') {
            when {
                branch 'master'
            }
            steps {
                echo 'Deploying master ...'
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
    }
}