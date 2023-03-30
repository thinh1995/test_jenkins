pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Merge code') {
            when {
                changeRequest target: 'master'
            }
            steps {
                sh 'git merge origin/dev'
            }
        }
    }
}