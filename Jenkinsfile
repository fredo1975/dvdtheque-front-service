pipeline {
    agent any
    tools {
        maven 'Maven 3.9.6'
        jdk 'jdk17'
    }
    environment {
    	def DEV_SERVER1_IP = '192.168.1.103'
    	def DEV_SERVER2_IP = '192.168.1.105'
    	def PROD_SERVER1_IP = '192.168.1.108'
    	def PROD_SERVER2_IP = '192.168.1.106'
    	def JAVA_OPTS='-Djava.io.tmpdir=/var/tmp/exportDir'
    	GIT_COMMIT_SHORT = sh(
                script: "printf \$(git rev-parse --short HEAD)",
                returnStdout: true
        )
        def GIT_BRANCH_NAME = getGitBranchName()
        def VERSION = getArtifactVersion(GIT_BRANCH_NAME,GIT_COMMIT_SHORT)
        def ARTIFACT = "dvdtheque-front-service-${VERSION}.jar"
        def TMDB_ARTIFACT = "dvdtheque-tmdb-service-${VERSION}.jar"
        def BATCH_ARTIFACT = "dvdtheque-batch-${VERSION}.jar"
        def ALLOCINE_ARTIFACT = "dvdtheque-allocine-service-${VERSION}.jar"
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "VERSION = ${VERSION}"
                    echo "PROD_SERVER1_IP = ${PROD_SERVER1_IP}"
                    echo "PROD_SERVER2_IP = ${PROD_SERVER2_IP}"
                    echo "DEV_SERVER1_IP = ${DEV_SERVER1_IP}"
                    echo "DEV_SERVER2_IP = ${DEV_SERVER2_IP}"
                    echo "VERSION = ${VERSION}"
                    echo "ARTIFACT = ${ARTIFACT}"
                    echo "TMDB_ARTIFACT = ${TMDB_ARTIFACT}"
                    echo "BATCH_ARTIFACT = ${BATCH_ARTIFACT}"
                    echo "ALLOCINE_ARTIFACT = ${ALLOCINE_ARTIFACT}"
                '''
            }
        }
        stage('Clone repository') {
			steps {
				script {
					checkout scm
				}
			}
		}
        stage('Build for development') {
        	when {
                branch 'develop'
            }
            steps {
		 		withMaven {
		 			sh """
			 			mvn -B org.codehaus.mojo:versions-maven-plugin:2.8.1:set -DprocessAllModules -DnewVersion=${VERSION}
			        	mvn -B clean compile
			      	"""
		    	}
		    }
        }
        stage('Build for production') {
        	when {
                branch 'master'
            }
            steps {
		 		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		 			sh """
		 				mvn -B org.codehaus.mojo:versions-maven-plugin:2.8.1:set -DprocessAllModules -DnewVersion=${VERSION}
				    	mvn -B clean compile
					"""
		    	}
		    }
        }
        stage('Unit Tests') {
        	when {
                branch 'develop'
            }
        	steps {
				withMaven(mavenSettingsConfig: 'MyMavenSettings') {
			 		script {
			 			sh ''' 
			 				mvn -B test -Darguments="${JAVA_OPTS}"
			 			'''
			 		}
	            }
			}
			post {
				always {
			    	junit '**/target/surefire-reports/*.xml'
			    }
			}   
        }
        stage('Deploy for development') {
            when {
                branch 'develop'
            }
            steps {
		 		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		 			script {
			 			sh """
					    	 mvn -B install -DskipTests
					    """
			 		}
		    	}
		    }
        }
        stage('Deploy for production') {
            when {
                branch 'master'
            }
            steps {
		 		withMaven(mavenSettingsConfig: 'MyMavenSettings') {
		 			script {
			 			sh """
					    	 mvn -B install -DskipTests
					    """
			 		}
		    	}
		    }
        }
        stage('Stopping Dev1 dvdtheque front service') {
        	when {
                branch 'develop'
            }
        	steps {
	       		sh 'ssh jenkins@$DEV_SERVER1_IP sudo systemctl stop dvdtheque-front-service.service'
	       	}
	    }
	    stage('Stopping Dev2 dvdtheque front service') {
	    	when {
                branch 'develop'
            }
        	steps {
	       		sh 'ssh jenkins@$DEV_SERVER2_IP sudo systemctl stop dvdtheque-front-service.service'
	       	}
	    }
	    stage('Stopping Prod1 dvdtheque front service') {
        	when {
                branch 'master'
            }
        	steps {
        		sh 'ssh jenkins@$PROD_SERVER1_IP sudo systemctl stop dvdtheque-front-service.service'
	       	}
	    }
	    stage('Stopping Prod2 dvdtheque front service') {
        	when {
                branch 'master'
            }
        	steps {
        		sh 'ssh jenkins@$PROD_SERVER2_IP sudo systemctl stop dvdtheque-front-service.service'
	       	}
	    }
	    
	    stage('Copying develop dvdtheque-front-service') {
	    	when {
                branch 'develop'
            }
            steps {
                script {
			 		sh """
			 			scp dvdtheque-rest-services/target/$ARTIFACT jenkins@${DEV_SERVER1_IP}:/opt/dvdtheque_rest_service/dvdtheque-rest-services.jar
			 		"""
			 		sh """
			 			scp dvdtheque-rest-services/target/$ARTIFACT jenkins@${DEV_SERVER2_IP}:/opt/dvdtheque_rest_service/dvdtheque-rest-services.jar
			 		"""
			 	}
            }
        }
        
        stage('Copying production dvdtheque-front-service') {
	    	when {
                branch 'master'
            }
            steps {
                script {
                	sh """
			 			scp dvdtheque-rest-services/target/$ARTIFACT jenkins@${PROD_SERVER1_IP}:/opt/dvdtheque_rest_service/dvdtheque-rest-services.jar
			 		"""
			 		sh """
			 			scp dvdtheque-rest-services/target/$ARTIFACT jenkins@${PROD_SERVER2_IP}:/opt/dvdtheque_rest_service/dvdtheque-rest-services.jar
			 		"""
			 	}
            }
        }
        
        stage('Sarting Dev1 Rest service') {
        	when {
                branch 'develop'
            }
        	steps {
	        	sh 'ssh jenkins@$DEV_SERVER1_IP sudo systemctl start dvdtheque-rest.service'
	        }
   		}
   		stage('Sarting Dev2 Rest service') {
   			when {
                branch 'develop'
            }
        	steps {
	        	sh 'ssh jenkins@$DEV_SERVER2_IP sudo systemctl start dvdtheque-rest.service'
	        }
   		}
   		stage('Sarting Prod1 Rest service') {
        	when {
                branch 'master'
            }
        	steps {
	        	sh 'ssh jenkins@$PROD_SERVER1_IP sudo systemctl start dvdtheque-rest.service'
	        }
   		}
   		stage('Sarting Prod2 Rest service') {
   			when {
                branch 'master'
            }
        	steps {
	        	sh 'ssh jenkins@$PROD_SERVER2_IP sudo systemctl start dvdtheque-rest.service'
	        }
   		}
   		stage('Sarting Prod2 Batch service') {
   			when {
                branch 'master'
            }
        	steps {
	        	sh 'ssh jenkins@$PROD_SERVER1_IP sudo systemctl start dvdtheque-batch.service'
	        }
   		}
   		stage('Check status Dev1 Rest service') {
   			when {
                branch 'develop'
            }
			steps {
				script {
				    sh 'ssh jenkins@$DEV_SERVER1_IP sudo systemctl status dvdtheque-rest.service'
			    }
			}
		}
		stage('Check status Dev2 Rest service') {
			when {
                branch 'develop'
            }
			steps {
				script {
				    sh 'ssh jenkins@$DEV_SERVER2_IP sudo systemctl status dvdtheque-rest.service'
			    }
			}
		}
		stage('Check status Prod1 Rest service') {
			when {
                branch 'master'
            }
			steps {
				script {
				    sh 'ssh jenkins@$PROD_SERVER1_IP sudo systemctl status dvdtheque-rest.service'
			    }
			}
		}
		stage('Check status Prod2 Rest service') {
			when {
                branch 'master'
            }
			steps {
				script {
				    sh 'ssh jenkins@$PROD_SERVER2_IP sudo systemctl status dvdtheque-rest.service'
			    }
			}
		}
    }
}

private String getGitBranchName(){
	def gitBranchName
	gitBranchName = env.BRANCH_NAME
	gitBranchName.trim()
}

private String getArtifactVersion(String gitBranchName,String gitCommit){
	if(gitBranchName == "develop"){
		return "${gitCommit}-SNAPSHOT"
	}
	if(gitBranchName == "master"){
		return "${gitCommit}"
	}
	return ""
}