node {
  checkout scm
  env.PATH = "${tool 'Maven3'}/bin:${env.PATH}"
  stage('Package') {
    dir('webapp') {
      sh 'mvn clean package -DskipTests'
    }
  }

  stage('Create Docker Image') {
    dir('webapp') {
        // docker.build("aa54035/docker-jenkins-pipeline:${env.BUILD_NUMBER}")
       sh label: '', script: "docker run --rm -u root -p 8080:8080 -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock  -v ${HOME}:/home  aa54035/docker-jenkins-pipeline:V11"
      
      //sh label: '', script: "sudo docker build -t aa54035/docker-jenkins-pipeline:${env.BUILD_NUMBER}"
      //by runing Sudo jenkins solded me that "with grate power comes great resposibility 
    }
  }

  stage ('Run Application') {
    try {
      // Start database container here
      // sh 'docker run -d --name db -p 8091-8093:8091-8093 -p 11210:11210 Aa54035/oreilly-couchbase:latest'

      // Run application using Docker image
      sh "DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`"
      sh "docker run -e DB_URI=$DB aa54035/docker-jenkins-pipeline:${env.BUILD_NUMBER}"

      // Run tests using Maven
      //dir ('webapp') {
      //  sh 'mvn exec:java -DskipTests'
      //}
    } catch (error) {
    } finally {
      // Stop and remove database container here
      //sh 'docker-compose stop db'
      //sh 'docker-compose rm db'
    }
  }

  stage('Run Tests') {
    try {
      dir('webapp') {
        sh "mvn test"
        docker.build("aa54035/docker-jenkins-pipeline:${env.BUILD_NUMBER}").push()
      }
    } catch (error) {

    } finally {
      junit '**/target/surefire-reports/*.xml'
    }
  }
}
