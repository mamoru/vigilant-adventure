#!groovy
// Jenkins Pipeline
pipeline {
  agent any
  /*options {
      parallelsAlwaysFailFast()
  }*/
  stages {
    stage('BuildAll') {
      parallel {
        
        stage('HelloWorld') {
          agent {
            docker { image 'gradle:jdk8' } // Use Jenkins Docker Plugin to run a new gradle container
          }
          stages {

            stage('HW:Build') {
              steps {
                sh "gradle build -s" // Use build.gradle directly using Gradle in docker container
                /* // Jenkins Pipelines Gradle step not available yet, using sh instead
                gradle {
                  tasks('build')
                  switches('-s') // stack-trace in case of build error
                  useWrapper(false)
                }*/
              } // steps
            } // stage HW:Build
            
            stage('HW:Archive') {
              steps {
                archiveArtifacts artifacts: 'build/libs/*', onlyIfSuccessful: true
              } // steps
            } // stage HW:Archive

            stage('HW:Run') {
              steps {
                sh "gradle run -s"
              } // steps
            } // stage HW:Run

          } // stages
        } // stage Hello-World

        stage('ElasticSearch') {
          agent {
            dockerfile {
              filename 'Dockerfile.ElasticSearch
              args '-v /var/run/docker.sock:/var/run/docker.sock'
            } // Use Jenkins Docker Plugin to run a new container from a dockerfile
          }
          stages {

            stage('ES:Fetch') {
              steps {
                checkout changelog: false, poll: false, \
                    scm: [$class: 'GitSCM', branches: [[name: '*/master']], \
                    doGenerateSubmoduleConfigurations: false, \
                    extensions: [[$class: 'CloneOption', noTags: true, reference: '', shallow: true, timeout: 30]], \
                    submoduleCfg: [], \
                    userRemoteConfigs: [[url: 'https://github.com/elastic/elasticsearch']]]
              } // steps
            } // stage ES:Fetch
            
            stage('ES:Build') {
              steps {
                sh "./gradlew assemble" // Use gradle wrapper in repository
              } // steps
            } // stage ES:Build

            stage('ES:Archive') {
              steps {
                archiveArtifacts artifacts: 'build/distributions/*', onlyIfSuccessful: true
              } // steps
            } // stage ES:Archive

          } // stages
        } // stage ElasticSearch

      } // parallel
    } // stage BuildAll
  } // stages
} // pipeline
