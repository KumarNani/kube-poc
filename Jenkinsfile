pipeline {
	 agent {
		  label "kube-slave"
			 }
	 parameters {
			choice choices: ['promote', 'rollback','build'], description: 'Task to do', name: 'task'
			choice choices: ['prod', 'dev','na', 'stag'], description: 'Environment for Deploy the App', name: 'environment'
			string (defaultValue: "00", description: 'Deployment Version applicable for promote job only', name: 'version')
			choice choices: ['master', 'dev'], description: 'git branchname', name: 'gitbranch'
			
			
		}	
	


    stages {

        
			stage ('checkout code') {
				steps {
					container(name: 'maven', shell: '/bin/bash') {
						git branch: '${gitbranch}',
						credentialsId: 'githublogin',
						url: 'https://github.com/techmartguru/kube-poc'
							  
				
			}               
			}
			}
			stage("Build displayName for dev") {
				when {
					expression { 
						// build and push when branch "dev" is requested
						
						params.gitbranch == 'dev'
					}
						
				}
				steps {
					container(name: 'maven', shell: '/bin/bash') {
						script {
							
							currentBuild.description = "snapshot-1.1"
                }
            }
			}
			}
			stage("Build displayName for master") {
				when {
					expression { 
						// build and push when branch "dev" is requested
						
						params.gitbranch == 'master'
					}
						
				}
				steps {
					container(name: 'maven', shell: '/bin/bash') {
						script {
							
							currentBuild.description = "release-1.1"
                }
            }
			}
			}
			stage ('Connect to GKE and GCR ') {
            steps {
			container(name: 'maven', shell: '/bin/bash') {
			//login on gke cluster
            sh 'gcloud auth activate-service-account --key-file /secrets/poc-kube-234001-9595fdb47852.json'
		    sh 'gcloud container clusters get-credentials kube-poc --zone us-central1-a --project poc-kube-234001'
			//login on registry
			sh 'cat /secrets/poc-kube-234001-9595fdb47852.json | docker login -u _json_key --password-stdin https://gcr.io'
            }
                
        }
		}	
			stage('Compile Using Maven for dev') {
					when {
					expression { 
						// build and push when branch "dev" is requested
						params.task == 'build' 
						
					}
						
				}
					steps {
					container(name: 'maven', shell: '/bin/bash') {
						
						sh ("mvn clean package -DskipTests")
						
						//build and push docker image to gcr
						sh 'docker build -t gcr.io/poc-kube-234001/hello-docker:${BUILD_NUMBER} .'
						sh 'docker push gcr.io/poc-kube-234001/hello-docker:${BUILD_NUMBER}'
						//deploy the newly created image to dev environment
						sh 'sed -i s/hello-docker:10/hello-docker:${BUILD_NUMBER}/g test-app.yaml'
						sh 'kubectl apply -f test-app.yaml -n dev'
						
					}
				}
				}
				
			stage('Maven Package') {
					steps {
					container(name: 'maven', shell: '/bin/bash') {
						sh 'echo "Hi This is testing"'
						
					}
				}		
			}
			stage('promote the application') {
				when {
					expression { 
						// build and push when environment is requested
						
						params.task == 'promote'
					}
					}
				steps {
					
					container(name: 'maven', shell: '/bin/bash') {
						
						sh 'sed -i s/hello-docker:10/hello-docker:${version}/g test-app.yaml'
						sh 'kubectl apply -f test-app.yaml -n ${environment}'
						
					}	
					
					}
				}
				stage('rollback on application') {
				when {
					expression { 
						// build and push when environment is requested
						
						params.task == 'rollback'
					}
					}
				steps {
					
					container(name: 'maven', shell: '/bin/bash') {
						
						sh 'kubectl rollout history deployment/test -n ${environment}'
						sh 'kubectl rollout undo deployment/test -n ${environment}'
						
					}	
					
					}
				}
			}
		}	
	
