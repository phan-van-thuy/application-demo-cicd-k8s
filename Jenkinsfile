pipeline {
  agent { label 'agent-1' }
  environment {
    REGISTRY           = credentials('docker-registry')
    USER               = credentials('docker-username')
    PASSWORD           = credentials('docker-password')
    SONAR_TOKEN        = credentials('sonar-token')
    KUBECONFIG_CONTENT = credentials('kubeconfig-b64')
    KUBECONFIG         = "kubeconfig"
  }

  stages {
    stage('Tag Image') {
      steps {
        script {
          env.TAG_LATEST = "${env.REGISTRY}/${env.GIT_BRANCH}:latest"
          env.TAG_COMMIT = "${env.REGISTRY}/${env.GIT_BRANCH}:${env.GIT_COMMIT.take(7)}"
        }
      }
    }

    stage('Test Registry Login') {
      steps {
        sh '''
          docker login -u $USER -p $PASSWORD $REGISTRY
        '''
      }
    }
    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv('sonar') {
          sh '''
            sonar-scanner \
              -Dsonar.projectKey=DEMO \
              -Dsonar.host.url=https://sonarque.demo \
              -Dsonar.login=$SONAR_TOKEN \
              -Dsonar.qualitygate.wait=false
          '''
        }
      }
    }

    stage('Build Image') {
      when {
        expression {
          return env.GIT_BRANCH == "origin/master"
        }
      }
      steps {
        sh '''
          docker login -u $USER -p $PASSWORD $REGISTRY
          docker build -f docker/Dockerfile -t $TAG_COMMIT -t $TAG_LATEST .
          docker push $TAG_COMMIT
          docker push $TAG_LATEST
        '''
      }
    }

    stage('Deploy to K8s') {
      when {
        expression {
          return env.GIT_BRANCH == "origin/master"
        }
      }
      steps {
        sh '''
          echo "$KUBECONFIG_CONTENT" | base64 --decode > $KUBECONFIG
          
          sed -i 's|IMAGE|'"$TAG_LATEST"'|g' docker/deployment.yaml
          kubectl apply -f docker/deployment.yaml
          if ! kubectl rollout status deployment/demo-deployment -n demo --timeout=60s; then
            echo "Deployment failed, rolling back..."
            kubectl rollout undo deployment/demo-deployment -n demo
            exit 1
          fi
          sleep 60
          kubectl get pod -n demo
        '''
      }
    }
  }
}
