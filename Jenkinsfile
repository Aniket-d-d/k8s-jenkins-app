pipeline{
    agent { label "dev-server"}  //defining agent

    stages{
        stage('app-code'){
            steps{
                
                dir('app-code') {
                     git url: "https://github.com/Aniket-d-d/ak-flask-docker.git", branch: "master" //clone-the app-code
                }
            }
        }
        
        stage('Build-and-Push-image '){
            environment {
                DOCKERHUB_CREDS = credentials('dockerHub')   //docker-credentials
            }
            steps{
			    dir('app-code'){
			     
                    sh "docker build -t vasecomappimage:v${BUILD_NUMBER} ." //building image on agent.
                    echo "Image Built Successfully"
                
                    sh "docker login -u $DOCKERHUB_CREDS_USR -p $DOCKERHUB_CREDS_PSW"
                    sh "docker tag vasecomappimage:v${BUILD_NUMBER} $DOCKERHUB_CREDS_USR/flaskapp:v${BUILD_NUMBER}"
                    sh "docker push $DOCKERHUB_CREDS_USR/flaskapp:v${BUILD_NUMBER}" //pushing image to docker-hub
                    echo "Images Pushed Successfully."
			    }
            }
            
        }
        
        stage('manifest-code'){
            steps{
                dir('manifest-code'){
                    git url: "https://github.com/Aniket-d-d/k8s-jenkins-app.git", branch: "main" //clone manifest-files
                }
            }
        }
        
        stage('create-secret'){
            environment {
                DOCKERHUB_CREDS = credentials('dockerHub')
            }
            steps{
                dir('manifest-code'){
                    sh "kubectl create secret docker-registry my-docker-secret --docker-username=$DOCKERHUB_CREDS_USR --docker-password=$DOCKERHUB_CREDS_PSW --docker-email=devopsengineer400@gmail.com"  //creating a k8s secret to pull app image from dockerhub
                }
            }
        }
        
		
		
		stage('deploy-app'){
            steps{
                dir('manifest-code'){
				            //applying all the manifest-files
				            sh "sed -i 's/build_no/v${BUILD_NUMBER}/g' flask-app-pod.yaml"  //change the image tag version on flask-app-pod manifest file as per the buold number.
                    sh "kubectl apply -f database-pod.yaml"
                    sh "kubectl apply -f database-service.yaml"
                    sh "kubectl apply -f flask-app-pod.yaml"
                    sh "kubectl apply -f flask-app-service.yaml"
                }
            }
        }
        
        stage('sleep-stage') {
            steps {
                    //waiting for pod deployment.
                    sleep(time: 60, unit: 'SECONDS')
            }
        }
        
    }
}
