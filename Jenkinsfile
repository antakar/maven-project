pipeline {
    agent any
 
    parameters {
         string(name: 'tomcat_dev', defaultValue: '3.8.170.252', description: 'Staging Server')
         string(name: 'tomcat_prod', defaultValue: '18.130.239.214', description: 'Production Server')
    }

    triggers {
        pollSCM('* * * * *')
    }

    tools {
        maven 'localMaven'
    }
 
 
stages{
    stage ('Builkd and Checkstyle'){
        parallel{
            stage('Build'){
                steps {
                    sh 'mvn clean package'
                }
                post {
                    success {
                        echo 'Now Archiving...'
                        archiveArtifacts artifacts: '**/target/*.war'
                    }
                }
            }
            stage('Run Static Analysis'){
                steps {
                    build job: 'static-analysis'
                }
            }
        }
    }
    
    stage ('Deployments'){
        parallel{
            stage ('Deploy to Staging'){
                steps {
                    sh "scp -i /home/jacek/.ssh/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_dev}:/var/lib/tomcat8/webapps"
                }
            }

            stage ("Deploy to Production"){
                steps {
                    sh "scp -i /home/jacek/.ssh/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_prod}:/var/lib/tomcat8/webapps"
                }
            }
        }
    }
}