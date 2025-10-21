pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
      stage('Unit Tests') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }  

      stage('SonarQube - SAST') {
          steps{
            sh "mvn clean verify sonar:sonar \
  -Dsonar.projectKey=numeric-application \
  -Dsonar.projectName='numeric-application' \
  -Dsonar.host.url=http://lubos-devops.westeurope.cloudapp.azure.com:9000 \
  -Dsonar.token=sqp_3d70779fd0433b71969f35c9da08f0212cfdbd14"
          }
      }
        
      stage('Docker Build and Push'){
        steps {
          withDockerRegistry(credentialsId: "docker-hub", url: "") {
              sh 'printenv'
              sh 'docker build -t xkolumber/numeric-app:""${GIT_COMMIT}"" .'
              sh 'docker push xkolumber/numeric-app:""${GIT_COMMIT}""'
            }
        
        }
      }
      stage('Kubernetes Deployment - DEV'){
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#xkolumber/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
            }
        }
      }

    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: 'target/pit-reports/**/mutations.xml'
        }
      }
    }
    }
}