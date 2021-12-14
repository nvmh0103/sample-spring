pipeline {
  agent any
   environment {
        PROJECT_ID = 'kubernetes-331801'
        LOCATION = 'asia-southeast1-a'
        CREDENTIALS_ID = 'Kubernetes'
        CLUSTER_NAME = 'ltat'          
    }

  tools {
    jdk 'jdk-11'
    maven 'mvn-3.6.3'
  }
  
  
  stages {
    stage('Build') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn package'
        }
      }
    }

    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn dependency-check:check'
        }

      }
    }

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv(installationName: 'sq1') {
          withMaven(maven : 'mvn-3.6.3') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }

    stage('Create and push container') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          withMaven(maven : 'mvn-3.6.3') {
            sh "mvn jib:build"
          }
        }
      } 
    }

//     stage('Anchore analyse') {
//       steps {
//         writeFile file: 'anchore_images', text: 'docker.io/nvmh0103/spring-boot-demo'
//         anchore name: 'anchore_images'
//       }
//     }

     stage('Deploy to GKE') {
      steps {
       step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'k8s.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
       }
     }
  }
}
