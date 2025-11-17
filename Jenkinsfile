pipeline {
  agent any

  environment {
    NEXUS_URL = 'http://localhost:8081/repository/maven-releases/'
    REPO_ID   = 'nexus'
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
        sh 'ls -l target || true'
      }
    }

    stage('Resolve coords & jar') {
      steps {
        script {
          GROUP = sh(script: "mvn help:evaluate -Dexpression=project.groupId -q -DforceStdout", returnStdout: true).trim()
          ARTIFACT = sh(script: "mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout", returnStdout: true).trim()
          VERSION = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
          JAR = sh(script: "ls target/*.jar | head -n1", returnStdout: true).trim()
          if (!GROUP || !ARTIFACT || !VERSION) { error "Failed to resolve Maven coords." }
          if (!JAR) { error "No JAR found in target/" }
          env.PROJECT_GROUP = GROUP
          env.PROJECT_ARTIFACT = ARTIFACT
          env.PROJECT_VERSION = VERSION
          env.BUILT_JAR = JAR
          echo "Will deploy: ${GROUP}:${ARTIFACT}:${VERSION} -> ${JAR}"
        }
      }
    }

    stage('Deploy to Nexus') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          sh '''
            mvn -B deploy:deploy-file \
              -Durl=${NEXUS_URL} \
              -DrepositoryId=${REPO_ID} \
              -Dfile=${BUILT_JAR} \
              -DgroupId=${PROJECT_GROUP} \
              -DartifactId=${PROJECT_ARTIFACT} \
              -Dversion=${PROJECT_VERSION} \
              -Dpackaging=jar \
              -Dusername=${NEXUS_USER} \
              -Dpassword=${NEXUS_PASS}
          '''
        }
      }
    }
  }

  post { success { echo "Deployed ${PROJECT_GROUP}:${PROJECT_ARTIFACT}:${PROJECT_VERSION}" } }
}
