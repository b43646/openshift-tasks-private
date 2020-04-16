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
    git url: 'http://gogs-xyz-gogs.apps.cluster-shanghai-3a4b.shanghai-3a4b.example.opentlc.com/CICDLabs/openshift-tasks-private.git'
  }

  // Using Maven build the war file
  // Do not run tests in this step
  stage('Build jar') {
      
    echo "Start Building"
    sh "${mvnCmd} clean package -DskipTests"

  }

  // Using Maven run the unit tests
  stage('Unit Tests') {
    echo "Running Unit Tests"
    //sh "${mvnCmd} test"
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
    echo "Building OpenShift container image task"
	sh "oc new-project hello"
	sh "oc new-build --binary=true --name=task redhat-openjdk18-openshift:1.5 -n hello"
    sh "oc start-build task --follow --from-file=./target/demo-docker-0.0.1-SNAPSHOT.jar -n hello"
  }

  // Deploy the built image to the Development Environment.
  stage('Deploy') {
    echo "Deploying container image to Development Project"
    sh "oc new-app --docker-image=image-registry.openshift-image-registry.svc:5000/hello/task --name=task --allow-missing-images"
	sh "oc expose dc task --port=8080"
	sh "oc expose svc/task"
	sh "sleep 60"
  }

  // Run Integration Tests in the Development Environment.
  stage('Integration Tests') {
    echo "Running Integration Tests"
    sh "curl `oc get route task -o jsonpath='{.spec.host}' -n hello`"
  }
}
