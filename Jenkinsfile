pipeline {
  agent any
  tools {
  maven 'maven'
  }
    stages {

  stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://dptrealtime@bitbucket.org/dptrealtime/webapp-cicd-integrations.git']]])
      }
   }
	  
	stage ('Build')  {
	    steps {
        dir('app'){
            sh "mvn package"
          }
        }    
   }
   
  stage ('SonarQube Analysis') {
    steps {
      withSonarQubeEnv('sonar') {           
				dir('app'){
          sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=stskey'
        }
    }
    }
 }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "https://dptdemo.jfrog.io/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "sts-libs-release-local",
                    snapshotRepo: "sts-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "default-maven-virtual",
                    snapshotRepo: "default-maven-virtual"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                    pom: 'app/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

stage('Docker Build') {
      steps {
        script {
              docker.withRegistry( 'https://registry.hub.docker.com', 'docker' ) {
              def customImage = docker.build("dpthub/webapp")
              customImage.push()
          }
      }
    }
}
   stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

  stage('Build Helm Charts') {
    steps {
        dir('charts') {
        withCredentials([usernamePassword(credentialsId: 'jfrog', usernameVariable: 'username', passwordVariable: 'password')]) {
             sh 'sudo /usr/local/bin/helm package webapp'
             sh 'sudo /usr/local/bin/helm push-artifactory webapp-1.0.tgz https://dptdemo.jfrog.io/artifactory/sts-helm-local --username $username --password $password'
		  }
        }
        }
      }

  } 
}