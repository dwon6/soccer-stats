#!groovy

// FULL_BUILD -> true/false build parameter to define if we need to run the entire stack for lab purpose only
final FULL_BUILD = params.FULL_BUILD
// HOST_PROVISION -> server to run ansible based on provision/inventory.ini
final HOST_PROVISION = params.HOST_PROVISION

final GIT_URL = 'https://github.com/dwon6/soccer-stats.git'
final NEXUS_URL = 'nexus.local:8081'

stage('Build') {
    node {
        git GIT_URL
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                def pom = readMavenPom file: 'pom.xml'
                sh "mvn -B versions:set -DnewVersion=${pom.version}-${BUILD_NUMBER}"
                sh "mvn -B -Dmaven.test.skip=true clean package"
                stash name: "artifact", includes: "target/soccer-stats-*.war"
        }
    }
}

stage('Unit Tests') {   
  node {
          withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
              sh "mvn -B clean test"
              stash name: "unit_tests", includes: "target/surefire-reports/**"
       }
   }
}

stage('Integration Tests') {
   node {
          withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
              sh "mvn -B clean verify -Dsurefire.skip=true"
              stash name: "it_tests", includes: 'target/failsafe-reports/**'
      }
  }
}

stage('Static Analysis') {
   node {
          withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
          withSonarQubeEnv('sonar'){
              unstash "it_tests"
              unstash "unit_tests"
              sh 'mvn sonar:sonar -DskipTests'
         }
      }
  }
}

stage('Approval') {
   timeout(time:3, unit:'DAYS') {
          input 'Do I have your approval for deployment?'
    }
}
