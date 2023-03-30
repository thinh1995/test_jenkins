pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Merge code') {
            when {
                changeRequest target: 'dev'
            }
            steps {
                sh 'git merge origin/dev'
            }
        }
    }
}