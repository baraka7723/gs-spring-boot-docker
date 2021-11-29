pipeline {
	agent any

	triggers {
		pollSCM 'H/10 * * * *'
	}

	options {
		disableConcurrentBuilds()
		buildDiscarder(logRotator(numToKeepStr: '14'))
	}

	stages {
		stage("checkout"){
			steps{
		checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/baraka7723/gs-spring-boot-docker']]])
		}
		}
		stage("build image"){
		sh 'docker build -t baraka7723/test:1.0.0 .'
		}
		
		stage("test: baseline (jdk8)") {
			agent {
				docker {
					image 'baraka7723/test:1.0.0'
					args '-v $HOME/.m2:/tmp/jenkins-home/.m2'
				}
			}
			options { timeout(time: 30, unit: 'MINUTES') }
			steps {
				sh 'test/run.sh'
			}
		}
		stage ("deploy"){
		                sh 'docker run -d  --rm -p 8081:8080 --name my-app baraka7723/test:1.0.0'
		}
	}
	

	post {
		changed {
			script {
				slackSend(
						color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
						channel: '#sagan-content',
						message: "${currentBuild.fullDisplayName} - `${currentBuild.currentResult}`\n${env.BUILD_URL}")
				emailext(
						subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
						mimeType: 'text/html',
						recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
						body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
			}
		}
	}
}
