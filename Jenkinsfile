pipeline {
    agent any
	tools {
		maven 'Maven'
	}
	
	environment {
		PROJECT_ID = 'buildpack-339211'
                CLUSTER_NAME = 'k8s-cluster'
                LOCATION = 'us-central1-c'
                CREDENTIALS_ID = 'kubernetes'		
	}
	
    stages {
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build') {
		    steps {
			    sh 'mvn clean package'
		    }
	    }
	    
	    stage('Test') {
		    steps {
			    echo "Testing..."
			    sh 'mvn test'
		    }
	    }
	    
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				    myimage = docker.build("msairam/devops:${env.BUILD_ID}")
			    }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
			    script {
				    echo "Push Docker Image"
				    withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
            				sh "docker login -u msairam -p ${dockerhub}"
				    }
				        myimage.push("${env.BUILD_ID}")
				    
			    }
		    }
	    }
	    
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' demochart/templates/serviceLB.yaml"
				sh "sed -i 's/tagversion/${env.BUILD_ID}/g' demochart/templates/deployment.yaml"
			    sh "echo Start deployment of serviceLB.yaml"
				sh "curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -"
				sh "sudo apt-get install apt-transport-https --yes"
				sh "echo deb https://baltocdn.com/helm/stable/debian/ all main | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list"
				sh "sudo apt-get update"
				sh "sudo apt-get install helm"
				sh "kubectl get pods"
				sh "kubectl config current-context"
				sh "kubectl config set-context \$(kubectl config current-context) --namespace=demo"
			    sh "helm upgrade --install helm-app demochart --set namespace=demo"
			    echo "Deployment Finished . .."
		    }
	    }
    }
}
