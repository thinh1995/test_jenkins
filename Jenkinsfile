void setBuildStatus(String message, String state) {
  step([
      $class: "GitHubCommitStatusSetter",
      reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/thinh1995/test_jenkins"],
      contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
      errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
      statusResultSource: [ $class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]] ]
  ]);
}

pipeline {
    agent {
        label 'ssh-agent'
    }

     stages {
        stage('Merge code master') {
            when {
                branch 'dev'
            }
            steps {
                sh 'git config --global user.email cuongthinhtuan2006@gmail.com'
                sh 'git config --global user.name thinh1995'
                sh 'git merge origin/master'
            }
        }

        stage('Merge code dev') {
            when {
                branch 'master'
            }
            steps {
                sh 'git config --global user.email cuongthinhtuan2006@gmail.com'
                sh 'git config --global user.name thinh1995'
                sh 'git merge origin/dev'
            }
        }
    }

    post {
        success {
            setBuildStatus("Build succeeded", "SUCCESS");
        }
        failure {
            setBuildStatus("Build failed", "FAILURE");
        }
    }
}