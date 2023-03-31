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
                echo "Current PR ID: ${env.CHANGE_ID}"
                echo "Current git branch ${env.GIT_BRANCH}"
                echo "Current PR State ${env.GITHUB_PR_STATE}"
                echo "PR Target branch ${env.GITHUB_PR_TARGET_BRANCH}"
                echo "PR Source branch ${env.GITHUB_PR_SOURCE_BRANCH}"
                // sh 'git config --global user.email cuongthinhtuan2006@gmail.com'
                // sh 'git config --global user.name thinh1995'
                // sh 'git fetch'
                // sh "git checkout ${env.GIT_BRANCH}"
                // sh 'git checkout master'
                // sh "git merge --no-edit ${env.GIT_BRANCH}"
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

                    pullRequest.addLabel('Build Passing')
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

                    pullRequest.addLabel('Build Failed')
                }
            }
        }
    }
}