import java.text.SimpleDateFormat
def TODAY = (new SimpleDateFormat("yyyyMMddHHmmss")).format(new Date())

pipeline {
	agent any
	environment {
		strDockerTag = "${TODAY}_${BUILD_ID}"
		strDockerImage ="hoyoungk12/cicd_guestbook:${strDockerTag}"
	}
		
	stages {
		stage('Checkout') {
		steps {
		    git branch: 'master', url:'https://github.com/sodlfma/guestbook.git'
			}
		}
		
		stage('Build') {
            steps {
                sh "./mvnw -Dmaven.test.failure.ignore=true clean package"
            }
            post {
                success {
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
		
		stage('Docker Image Build') {
			steps {
				script {
				    oDockImage = docker.build(strDockerImage, "--build-arg VERSION=${strDockerTag} -f Dockerfile .")
					docker.withRegistry('', 'DockerHub_ho') {
					    oDockImage.push()
				    }
				}
			}
		}
		
		stage('SSH Staging Server') {
            steps {
                sshagent(['Staging-PrivateKey']) {
                // some block
                
                sh "ssh -o StrictHostKeyChecking=no root@3.97.14.245 docker container rm -f guestbookapp"
                sh "ssh -o StrictHostKeyChecking=no root@3.97.14.245 docker container run \
                	-d \
                	-p 38080:80 \
                	--name=guestbookapp \
                	-e MYSQL_IP=3.99.192.194 \
                	-e MYSQL_PORT=3306 \
                	-e MYSQL_DATABASE=guestbook \
                	-e MYSQL_USER=root \
                	-e MYSQL_PASSWORD=education \
                	${strDockerImage} "
                }
            }
        }
        
        stage ('JMeter LoadTest') {
            steps {
                sh '~/lab/sw/jmeter/bin/jmeter.sh -j jmeter.save.saveservice.output_format=xml -n -t src/main/jmx/guestbook_loadtest.jmx -l loadtest_result.jtl'
                perfReport filterRegex: '', showTrendGraphs: true, sourceDataFiles: 'loadtest_result.jtl'
            }
        }

        stage("Slack Notification") {
            steps {
                slackSend(tokenCredentialId: 'slack-token'
                    , channel: '#일반'
                    , color: 'good'
                    , message: '교육 채널 메세지')
            }
        }
        
	}
}

