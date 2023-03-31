pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Test') {
             when {
                changeRequest()
                
            }
            steps {
                script {
                    echo "Current PR ID: ${env.CHANGE_ID}"
                    echo "Current git branch ${env.GIT_BRANCH}"
                    echo "Current PR State ${pullRequest.state}"
                    echo "PR Target branch ${pullRequest.base}"
                    echo "PR Source branch ${pullRequest.headRef}"
                    echo "PR can merge ${pullRequest.mergeable}"

                    // if (!pullRequest.mergeable) {
                    //     throw new Exception("PR has conflicts!")
                    // }

                    git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
                    git fetch --all
                    git checkout "${pullRequest.base}"
                    git merge --no-edit "origin/${pullRequest.headRef}"
                }
            }
        }
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
    }
}