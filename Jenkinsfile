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
                        def mvnHome = tool 'localMaven'
                
                        sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs spotbugs:spotbugs"
                
                        def checkstyle = scanForIssues tool: [$class: 'CheckStyle'], pattern: '**/target/checkstyle-result.xml'
                        publishIssues issues:[checkstyle]
                    
                        def pmd = scanForIssues tool: [$class: 'Pmd'], pattern: '**/target/pmd.xml'
                        publishIssues issues:[pmd]
                        
                        def cpd = scanForIssues tool: [$class: 'Cpd'], pattern: '**/target/cpd.xml'
                        publishIssues issues:[cpd]
                        
                        def findbugs = scanForIssues tool: [$class: 'FindBugs'], pattern: '**/target/findbugsXml.xml'
                        publishIssues issues:[findbugs]
                
                        def spotbugs = scanForIssues tool: [$class: 'SpotBugs'], pattern: '**/target/spotbugsXml.xml'
                        publishIssues issues:[spotbugs]
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
    