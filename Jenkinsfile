pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Test') {
             when {
                changeRequest target: 'master'
                
            }
            steps {
                echo "Current Pull Request ID: ${env.CHANGE_ID}"
                echo "Current git branch ${env.GIT_BRANCH}"
                sh 'git config --global user.email cuongthinhtuan2006@gmail.com'
                sh 'git config --global user.name thinh1995'
                sh 'git fetch'
                sh "git checkout ${env.GIT_BRANCH}"
                sh 'git checkout master'
                sh "git merge --no-edit ${env.GIT_BRANCH}"
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

                    // pullRequest.review('APPROVE', 'Good')
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

                    // pullRequest.review('CHANGES_REQUESTED', 'Change is the essential process of all existence.')
                }
            }
        }
    }
}