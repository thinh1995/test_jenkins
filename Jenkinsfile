pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Merge code') {
            when {
                branch 'dev'
            }
            steps {
                sh 'git merge origin/master'
            }
        }
    }
}