@Library('dynatrace@master') _

pipeline {
  parameters {
    string(name: 'SERVICE', defaultValue: 'carts', description: 'Name of the service')
    /* string(name: 'VERSION_NUMBER', defaultValue: '0.1.0', description: 'Version number to be released') */
  }
  agent {
    label 'kubegit'
  }
  environment {
    APP_NAME = "${env.SERVICE}"
  }
  stages {
    stage('Increase version number') {
      steps {
        container('git') {
          withCredentials([usernamePassword(credentialsId: 'git-credentials-acm', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "git config --global user.email ${env.GITHUB_USER_EMAIL}"
            sh "git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/${env.SERVICE}"
            sh "version=`cat ${env.SERVICE}/version`"
            sh "cd ${env.SERVICE}/ && git checkout -b release/$version"
            sh "cd ${env.SERVICE}/ && git push --set-upstream https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/${env.SERVICE} release/$version"
            sh "cd ${env.SERVICE}/ && git checkout master"
            sh "minor_version=`cat ${env.SERVICE}/version | cut -d'.' -f2`"
            sh "new_minor_version=$((minor_version+1))"
            sh "cd ${env.SERVICE}/ && sed -i 's/\.'\"$minor_version\"'\..*/\.'\"$new_minor_version\"'\.0/' version"
            sh "cd ${env.SERVICE}/ && git add version"
            sh "cd ${env.SERVICE}/ && git commit -am 'Increased version to ${env.VERSION_NUMBER}'"
            sh "cd ${env.SERVICE}/ && git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${env.GITHUB_ORGANIZATION}/${env.SERVICE}"
          } 
        }
      }
    }
    /*
    stage('Scan multibranch pipeline') {
      steps {
        build job: "sockshop/{env.SERVICE} multibranch/release%2F${env.VERSION_NUMBER}",
          parameters: []
      }
    }
    */
  }
}
