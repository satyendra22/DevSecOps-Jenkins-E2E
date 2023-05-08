pipeline {
  agent any
  tools {
    maven 'Maven_3_9_1'
  }

  stages {
     stage('Compile_SAST_SonarAnalysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          bat("mvn -Dmaven.test.failure.ignore verify sonar:sonar -Dsonar.login=$SONAR_TOKEN -Dsonar.projectKey=easybuggy -Dsonar.host.url=http://localhost:9000/")
        }
      }
    }
    
   stage('Cred_truffleHog') {
      steps {
        script {
            try {
                  bat("docker run -v ${WORKSPACE}:/pwd trufflesecurity/trufflehog:latest github --org=trufflesecurity")
              } catch (err) {
              echo err.getMessage() 
            }
          }
      }
    }
    stage('SAST_Semgrep') {
      steps {
        script {
            try {
                  bat("docker run -v ${WORKSPACE}:/src --workdir /src returntocorp/semgrep-agent:v1 semgrep-agent --config auto")
              } catch (err) {
              echo err.getMessage() 
            }
          }
      }
    }
    
    stage('SBOM_CycloneDX') {
      steps {
        script {
            try {
                  bat("docker run -v ${WORKSPACE}:/app:rw -t ghcr.io/cyclonedx/cdxgen -r /app -o /app/bom.json")
              } catch (err) {
              echo err.getMessage() 
            }
          }
      }
    }
    
    stage('Docker_Build') {
      steps {
        withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
          script {
            try {
              app = docker.build("satyendra22/testeb")
            } catch (err) {
              echo err.getMessage() 
            }
          }
        }
      }
    }
    stage('ContainerScan_Snyk') {
      steps {
        withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
          script {
            try {
              bat("C:\\DevSecOps-Demo\\snyk\\snyk-win.exe  container test satyendra22/testeb")
            } catch (err) {
              echo err.getMessage()
            }
          }
        }
      }
    }
    stage('SCA_Snyk') {
      steps {
        withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
          bat("mvn snyk:test -fn")
        }
      }
    }
    
    stage('SCA_DepCheck') {
      steps {
        bat("C:\\DevSecOps-Demo\\dependency-check-8.2.1-release\\dependency-check\\bin\\dependency-check.bat --scan ${WORKSPACE}\\target\\easybuggy-1-SNAPSHOT\\WEB-INF\\lib")
      }
    }
    stage('Build and Run container image locally ') {
      steps {
        script {
          try {
            bat ("docker build . -t sc2:local")
            bat ("docker run -d -p 8080:8080 sc2:local")
          } catch (err) {
              echo err.getMessage()
          }
        }
      }
    }
    stage('DAST_ZAP') {
      steps {
        bat("C:\\DevSecOps-Demo\\ZAP_2.12.0_Crossplatform\\ZAP_2.12.0\\zap.sh -port 9393 -cmd -quickurl http://localhost:8080 -quickprogress -quickout C:\\DevSecOps-Demo\\ZAP_2.12.0_Crossplatform\\ZAP_2.12.0\\Output.html")
      }
    }


    stage('IAC_checkov') {
      steps {
        bat("checkov -s -f main.tf")
      }
    }
    stage('IAC_KICKS') {
      steps {
        script {
          try {
            bat ("docker run -v ${WORKSPACE}:/path checkmarx/kics:latest scan -p /path -o /path/results --report-formats html")
          } catch (err) {
              echo err.getMessage()
          }
        }
      }
    }
  }
}
