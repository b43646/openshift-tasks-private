#!groovy

// Run this pipeline on the custom Maven Slave ('maven-appdev')
// Maven Slaves have JDK and Maven already installed
// 'maven-appdev' has skopeo installed as well.
node('maven') {
  // Define Maven Command. Make sure it points to the correct
  // settings for our Nexus installation (use the service to
  // bypass the router). The file nexus_openshift_settings.xml
  // needs to be in the Source Code repository.
  // def mvnCmd = "mvn -s ./nexus_openshift_settings.xml"
  def mvnCmd = "mvn"

  // Checkout Source Code
  stage('Checkout Source') {
    git credentialsId: 'test', url: 'http://gogs-xyz-gogs.apps.cluster-shanghai-3a4b.shanghai-3a4b.example.opentlc.com/CICDLabs/openshift-tasks-private.git'
  }

  // The following variables need to be defined at the top level
  // and not inside the scope of a stage - otherwise they would not
  // be accessible from other stages.
  // Extract version and other properties from the pom.xml
  def groupId    = getGroupIdFromPom("pom.xml")
  def artifactId = getArtifactIdFromPom("pom.xml")
  def version    = getVersionFromPom("pom.xml")

  // Set the tag for the development image: version + build number
  def devTag  = "${version}-${BUILD_NUMBER}"
  // Set the tag for the production image: version
  def prodTag = "${version}"

  // Using Maven build the war file
  // Do not run tests in this step
  stage('Build war') {
      
    echo "Building version ${devTag}"
    sh "${mvnCmd} clean package -DskipTests"

  }

  // Using Maven run the unit tests
  stage('Unit Tests') {
    echo "Running Unit Tests"
    sh "${mvnCmd} test"
  }

  // Using Maven call SonarQube for Code Analysis
  stage('Code Analysis') {
    echo "Running Code Analysis"
    // TBD
  }

  // Publish the built war file to Nexus
  stage('Publish to Nexus') {
    echo "Publish to Nexus"
    // TBD
  }

  // Build the OpenShift Image in OpenShift and tag it.
  stage('Build and Tag OpenShift Image') {
    echo "Building OpenShift container image tasks:${devTag}"
    sh "oc start-build tasks --follow --from-file=./target/openshift-tasks.war -n loren-tasks-dev"
    openshiftTag alias: 'false', destStream: 'tasks', destTag: devTag, destinationNamespace: 'loren-tasks-dev', namespace: 'loren-tasks-dev', srcStream: 'tasks', srcTag: 'latest', verbose: 'false'
  }

  // Deploy the built image to the Development Environment.
  stage('Deploy to Dev') {
    echo "Deploying container image to Development Project"
    
    // Update the Image on the Development Deployment Config
    sh "oc set image dc/tasks tasks=docker-registry.default.svc:5000/loren-tasks-dev/tasks:${devTag} -n loren-tasks-dev"

    // Update the Config Map which contains the users for the Tasks application
    sh "oc delete configmap tasks-config -n loren-tasks-dev --ignore-not-found=true"
    sh "oc create configmap tasks-config --from-file=./configuration/application-users.properties --from-file=./configuration/application-roles.properties -n loren-tasks-dev"

    // Deploy the development application.
    openshiftDeploy depCfg: 'tasks', namespace: 'loren-tasks-dev', verbose: 'false', waitTime: '', waitUnit: 'sec'
    openshiftVerifyDeployment depCfg: 'tasks', namespace: 'loren-tasks-dev', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: '', waitUnit: 'sec'
    openshiftVerifyService namespace: 'loren-tasks-dev', svcName: 'tasks', verbose: 'false'
  }

  // Run Integration Tests in the Development Environment.
  stage('Integration Tests') {
    echo "Running Integration Tests"
    sleep 15

    // Create a new task called "integration_test_1"
    echo "Creating task"
    sh "curl -i -u 'tasks:redhat1' -H 'Content-Length: 0' -X POST http://tasks.loren-tasks-dev.svc.cluster.local:8080/ws/tasks/integration_test_1"

    // Retrieve task with id "1"
    echo "Retrieving tasks"
    sh "curl -i -u 'tasks:redhat1' -H 'Content-Length: 0' -X GET http://tasks.loren-tasks-dev.svc.cluster.local:8080/ws/tasks/1"

    // Delete task with id "1"
    echo "Deleting tasks"
    sh "curl -i -u 'tasks:redhat1' -H 'Content-Length: 0' -X DELETE http://tasks.loren-tasks-dev.svc.cluster.local:8080/ws/tasks/1"
  }
}

// Convenience Functions to read variables from the pom.xml
// Do not change anything below this line.
// --------------------------------------------------------
def getVersionFromPom(pom) {
  def matcher = readFile(pom) =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}
def getGroupIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<groupId>(.+)</groupId>'
  matcher ? matcher[0][1] : null
}
def getArtifactIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<artifactId>(.+)</artifactId>'
  matcher ? matcher[0][1] : null
}
