pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Merge code') {
            when {
                branch 'master'
                changeRequest targetBranch: 'master'
            }
            steps {
                sh 'git merge origin/master'
            }
        }
    }
}