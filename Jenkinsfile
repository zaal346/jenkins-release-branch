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
            sh "git clone http://${GIT_USERNAME}:${GIT_PASSWORD}@${env.BASTION_IP}/${env.GITHUB_ORGANIZATION}/${env.SERVICE}"
            sh "version=`cat ${env.SERVICE}/version`"
            sh "cd ${env.SERVICE}/ && git checkout -b release/`cat version`"
            sh "cd ${env.SERVICE}/ && git push --set-upstream http://${GIT_USERNAME}:${GIT_PASSWORD}@${env.BASTION_IP}/${env.GITHUB_ORGANIZATION}/${env.SERVICE} release/`cat version`"
            sh "cd ${env.SERVICE}/ && git checkout master"
            sh "cp increment_version.sh ${env.SERVICE}/ && chmod +x ${env.SERVICE}/increment_version.sh"
            sh "cd ${env.SERVICE}/ && sh increment_version.sh"
            sh "cd ${env.SERVICE}/ && git add version"
            sh "cd ${env.SERVICE}/ && git commit -am 'Bumped up version'"
            sh "cd ${env.SERVICE}/ && git push http://${GIT_USERNAME}:${GIT_PASSWORD}@${env.BASTION_IP}/${env.GITHUB_ORGANIZATION}/${env.SERVICE}"
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
