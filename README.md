# kube-poc
Webhook testig now for prod.


/**
 * This pipeline will run a Docker image build
 */
environment {
    GOOGLE_APPLICATION_CREDENTIALS = credentials('dummy')
    no_proxy = '10.148.14.66'
}

def label = "docker-${UUID.randomUUID().toString()}"

podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: gcr.io/df-sre-tools-npe-fbfc/cloud-sdk
    env:
    - name: nexus3JarUrl
      value: ${params.nexus3JarUrl}
    - name: gcrTagVersion
      value: ${params.gcrTagVersion}
    - name: service
      value: ${params.service}
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
  ) {

properties([
        parameters([
                string(description: 'Nexus3 Jar url?', name: 'nexus3JarUrl'),
                string(description: 'GCR Tag Version?', name: 'gcrTagVersion'),
                choice(choices: ['il-elasticsearch-service', 'il-rest-api-service'], description: 'What Service?', name: 'service')
        ])
])

  def image = "gcr.io/testing/"
  node(label) {
    stage('Build Docker image') {
      git branch:'test1', url:'https://test@bitbucket.testing.com/scm/il/il-docker.git'
      container('docker') {
        withCredentials([file(credentialsId: 'dummy', variable: 'credentialsJson')]) {
             sh "cp \$credentialsJson /credentials.json"
        }
        sh 'echo nexus3JarUrl: ${nexus3JarUrl}'
        sh 'echo gcrTagVersion: ${gcrTagVersion}'
        sh 'echo service: ${service}'
        sh 'gcloud auth activate-service-account --key-file=/credentials.json'
        sh 'gcloud auth configure-docker --quiet'
		sh 'curl -O ${nexus3JarUrl}'
        sh "mv *.jar rest-api/"
		sh "docker build -t ${image}:${gcrTagVersion} --build-arg JAR_FILE=il-rest-api-1.3.0-20190116.203011-187.jar --build-arg PROJECT_ID=df-sre-tools-npe-fbfc rest-api/"
        sh "docker push ${image}"
      }
    }
  }
}
