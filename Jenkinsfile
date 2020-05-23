currentBuild.displayName = "${currentBuild.projectName}-${currentBuild.currentResult}#${currentBuild.number}"
serversMap = [dev: "172.31.32.179",stg: "172.31.55.12", prod: "172.31.53.219"]
pipeline{
    agent any
    parameters {
      choice choices: ['dev',  'stg', 'prod'], description: 'Choose the app env to deploy', name: 'deployenv'
      choice choices: ['develop', 'release', 'master'], description: 'Choose the branch to deploy', name: 'branchname'
    }
	environment {
        Tomcat_host_one = "ec2-user@172.31.32.179"
		Tomcat_host_two = "ec2-user@172.31.55.12"
		Tomcat_host_three = "ec2-user@172.31.53.219"
        Tomcat_svc = "/usr/sbin/service tomcat"
    }
	
    
    stages{
        stage('Maven Package'){
            steps{
                sh "git checkout ${params.branchname}"
                echo "We are building ${params.branchname} branch"
				sh script: 'mvn package'
            }
        }
        
        stage('tomcat-dev'){
            when{
                expression { params.deployenv == 'dev' }
            }
            steps{
                echo "We are deploying to dev environment with ip ${serversMap[params.deployenv]}"
                sshagent(['tomcat-dev']) {
                    sh "scp -o StrictHostKeyChecking=no  target/car-rentals*.war  ${Tomcat_host_one}:/opt/tomcat8/webapps/"
                    sh "ssh ${Tomcat_host_one} ${Tomcat_svc} stop"
                    sh "ssh ${Tomcat_host_one} ${Tomcat_svc} start"
                }
            }
        
        }
        stage('tomcat-test'){
            when{
                expression { params.deployenv == 'stg' }
            }
            steps{
                echo "We are deploying to staging environment with ip ${serversMap[params.deployenv]}"
                sshagent(['tomcat-test']) {
                    sh "scp -o StrictHostKeyChecking=no  target/car-rentals*.war  ${Tomcat_host_two}:/opt/tomcat8/webapps/"
                    sh "ssh ${Tomcat_host_two} ${Tomcat_svc} stop"
                    sh "ssh ${Tomcat_host_two} ${Tomcat_svc} start"
                }
            }
        
        }
        stage('tomcat-prod'){
            when{
                expression { params.deployenv == 'prod' }
            }
            steps{
                echo "We are deploying to production environment with ip ${serversMap[params.deployenv]}"
                sshagent(['tomcat-prod']) {
                    sh "scp -o StrictHostKeyChecking=no  target/car-rentals*.war  ${Tomcat_host_three}:/opt/tomcat8/webapps/"
                    sh "ssh ${Tomcat_host_three} ${Tomcat_svc} stop"
                    sh "ssh ${Tomcat_host_three} ${Tomcat_svc} start"
                }
            }
        
        }
    }
    
}
