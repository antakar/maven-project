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
        stage ('Build and Checkstyle'){
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
                //stage('Run Static Analysis'){
                    //steps {
                        //build job: 'static-analysis'
                    //}
                stage ('Analysis') {
                    steps{
                        [$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml', unstableTotalAll:'0']
                        }
                    steps{
                        [$class: 'PmdPublisher', pattern: '**/target/pmd.xml', unstableTotalAll:'0']
                        }
                    steps{
                        [$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml', unstableTotalAll:'0']
                        }
                }
                }
            }
        }

        stage ('Deployments'){
            parallel{
                stage ('Deploy to Staging'){
                    steps {
                        sh "scp -o StrictHostKeyChecking=No -i /var/lib/jenkins/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_dev}:/var/lib/tomcat8/webapps"
                    }
                }

                stage ("Deploy to Production"){
                    steps {
                        sh "scp -o StrictHostKeyChecking=No -i /var/lib/jenkins/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_prod}:/var/lib/tomcat8/webapps"
                    }
                }
            }
        }
    }
