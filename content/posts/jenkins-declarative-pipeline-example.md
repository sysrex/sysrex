---
title: "Jenkins declarative pipeline"
date: 2018-01-19T09:22:40+01:00
tags: ["Jenkins"]
summary: "Jenkins declarative pipeline example"
draft: false
---

### This is just a sample Jenkinsfile to get you started 

{{<highlight groovy>}}

pipeline {
	agent any
    options {
        ansiColor('xterm')
    }
	stages {
		stage ('Pull Code from Dev Branch') {
            steps {
			    checkout scm
            }
		}
		stage ('Test: integration-&-quality') {
            steps {
                sh 'sleep 10'
                sh 'echo "Integration Passed"'
            }
		}
		stage ('Test: functional') {
            steps {
                sh 'sleep 10'
                sh 'echo "Functional Passed"'
            }
		}
		stage ('Test: load-&-security') {
            steps {
                sh 'sleep 10'
                sh 'echo "Load and Security Passed"'
            }
		}
		stage ('Approval') {
            steps {
                sh 'sleep 10'
                sh 'echo "Approval Received"'
            }
		}
		stage ('Deploy:dev') { 
            steps {
                sh 'echo "Deployment to Dev Environment"'
                script {
                    sh """ssh -tt ubuntu@xx.xxx.xxx.xxx << EOF
                    lots of commands
                    exit
                    EOF"""
                }
            }
		}
	}
}

{{</highlight>}}