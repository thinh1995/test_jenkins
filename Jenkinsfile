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
                echo "Current PR State ${pullRequest.state}"
                echo "PR Target branch ${pullRequest.base}"
                echo "PR Source branch ${pullRequest.headRef}"
                echo "PR can merge ${pullRequest.mergeable}"
                // sh 'git config --global user.email cuongthinhtuan2006@gmail.com'
                // sh 'git config --global user.name thinh1995'
                git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
                git fetch --all
                sh "git checkout ${pullRequest.base}"
                sh "git merge --no-edit ${pullRequest.headRef}"
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

                    pullRequest.removeLabel('Build Failed')
                    pullRequest.addLabel('Build Success')
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

                    pullRequest.addLabel('Build Success')
                    pullRequest.addLabel('Build Failed')
                }
            }
        }
    }
}