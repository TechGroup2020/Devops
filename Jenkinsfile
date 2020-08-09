#!/usr/bin/env groovy
// Declartive Pipeline Scripting
// app, build dependents and env_properties_repo source code url properties 
app_repo =  'https://github.com/TechGroup2020/sourcecodeAPP.git'
build_depends_repo = 'https://github.com/TechGroup2020/Devops.git'

pipeline {
      agent { label 'jenkins-slave-java' }
      options {
         buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '90', numToKeepStr: '90')
         timestamps()
         skipStagesAfterUnstable()
      }

   stages {
      // Multiple Stage Sections Defined
      stage('cleanWorkSpace') {

            steps {
               cleanWs()
            }
      }
     
      stage('checkout-code') {
       
         steps {
               script {
                  
                  // checkout app-code
                  checkout([$class: 'GitSCM', branches: [[name: "*/master"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'e5297a36-afd0-4715-9874-689876141e47', url: app_repo]]])
                  // build-dependents-code checkout
				  checkout([$class: 'GitSCM', branches: [[name: "*/master"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'build-depends-code']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'e5297a36-afd0-4715-9874-689876141e47', url: build_depends_repo]]])
         }
      }


      stage('compile-code') {

         steps {
            sh '''
                cd "${WORKSPACE}"
                cp -rf "${WORKSPACE}"/build-depends-code/Dockerfile .
                #cp -rf "${WORKSPACE}"/build-depends-code/env.sh .
				cp -rf "${WORKSPACE}"/build-depends-code/application.yml .
               '''
         }
      } 

      stage ('build-docker-image') {


         steps {
            sh '''
            set +x
            
               #docker pull base images
               #sudo docker pull openjdk
              
               #build docker images for maps
               sudo docker build -t ecr-app .
            
               sudo docker tag ecr-app:latest 107747809792.dkr.ecr.ap-south-1.amazonaws.com/ecr-app:latest
               sudo docker tag 107747809792.dkr.ecr.ap-south-1.amazonaws.com/ecr-app:latest 107747809792.dkr.ecr.ap-south-1.amazonaws.com/ecr-app:${BUILD_NUMBER}

               # push docker image into dev environment
               sudo docker push 107747809792.dkr.ecr.ap-south-1.amazonaws.com/ecr-app:latest
               sudo docker push 107747809792.dkr.ecr.ap-south-1.amazonaws.com/ecr-app:${BUILD_NUMBER}
               
			   # docker workbench security
			   mkdir -p /tmp/bench
               git clone https://github.com/docker/docker-bench-security.git
               cd docker-bench-security
               echo
               echo "Scanning  ecr-app"
               sudo sh docker-bench-security.sh -b -l /tmp/bench/docker-bench-security.sh.log -i ecr-app
            
            '''
          	publishHTML target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '/tmp/bench',
                        reportFiles: 'docker-bench-security.sh.log',
                        reportName: 'docker-security-report'
                 ]
         }
      }
      
	}
     
      // Stages Sections is done
   } 
    post {
            failure {
                emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}',
                        to: EMAIL_TO,
                        subject: 'Build failed in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
            }
            unstable {
                emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}',
                        to: EMAIL_TO,
                        subject: 'Unstable build in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
            }
            changed {
                emailext body: 'Check console output at $BUILD_URL to view the results.',
                        to: EMAIL_TO,
                        subject: 'Jenkins build is back to normal: $PROJECT_NAME - #$BUILD_NUMBER'
            }

   }
}