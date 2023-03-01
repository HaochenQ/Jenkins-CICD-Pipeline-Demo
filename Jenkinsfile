
pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub-cred-howard')
	}

	stages {

		stage('Build') {

			steps {
				sh 'docker build -t howardq1997/reactapp:latest .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push howardq1997/reactapp:latest'
			}
		}
	}

	post {
		always {
			sh 'docker logout'
		}
	}

}