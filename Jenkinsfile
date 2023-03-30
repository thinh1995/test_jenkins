pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Merge code') {
            when {
                branch 'master'
                changeRequest target: 'master'
            }
            steps {
                sh 'git merge origin/master'
            }
        }
    }
}