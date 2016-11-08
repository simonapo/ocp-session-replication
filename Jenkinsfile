// Responsible for
//  1. Determining new version of a build
//  2. building application
//  3. executing Junit tests included in the app

node {
    stage 'Build'
    echo "Groovy:  Build stage:  ";
    // git url: 'https://github.com/wkulhanek/ocp-session-replication.git', credentialsId: ''
    // def version = getBuildVersion("pom.xml")
    // env.BUILD_VERSION = version
    // env.BUILD_GROUP_ID = getGroupIdFromPom("pom.xml")
    // env.BUILD_ARTIFACT_ID = getArtifactIdFromPom("pom.xml")
    // def branch = 'build-' + version
    // env.BUILD_BRANCH = branch
    //
    // sh "git checkout -b ${branch}"
    // sh "mvn -f pom.xml versions:set -DgenerateBackupPoms=false -DnewVersion=${version}"

    sh "mvn clean package -DskipTests"
    stash excludes: 'target/', includes: '**', name: 'source'

    // This is where integration tests would be executed
    stage 'Integrate'
    unstash 'source'
        // integrationTests()

    // This is where your built artifact would be pushed to a Nexus repo.
    // For the purpose of the openshift dev course, we'll skip this step because
    // a dedicated Nexus repository for each student is not provided
    stage 'Publish'
    unstash 'source'
        // publishToNexusAndCommitBranch(env.BUILD_VERSION, env.BUILD_BRANCH)


    // Publish to a QA environment
    // Uses `oc` utility of OSE to trigger S2I BuildConfig object in QA environment
    // Notification to members of the DevOps team should be sent out indicating that work flow has stopped at this human task
    stage 'QA'
    deployToJBossEAP("webapp-qa")

    // Wait until authorization to push to production
    stage 'Approve'
    timeout(time: 2, unit: 'DAYS') {
        input message: 'Do you want to deploy into production?'
    }

    // Push to production
    // Uses `oc` utility of OSE to trigger S2I BuildConfig object in prod environment
    stage 'Production'
    deployToJBossEAP("webapp-prod")
}


def integrationTests() {
    sh "mvn -f pom.xml verify"
}

def publishToNexusAndCommitBranch(version, branch) {
    sh "mvn -f pom.xml deploy -DaltDeploymentRepository=internal.nexus::default::http://nexus:8080/content/repositories/releases"
    def commit = "Build " + version
    sh "git add **/pom.xml && git commit -m \"${commit}\" && git push origin ${branch}"
}

def deployToJBossEAP(buildConfig) {
    echo "PROJECT = $PROJECT"
    echo "BUILD_CONFIG = ${buildConfig}"
    openshiftBuild apiURL: '', authToken: '', bldCfg: buildConfig, checkForTriggeredDeployments: 'false', namespace: PROJECT, verbose: 'false'
    openshiftVerifyBuild apiURL: '', authToken: '', bldCfg: buildConfig, checkForTriggeredDeployments: 'false', namespace: PROJECT, verbose: 'false'
}

def getVersionFromPom(pom) {
    def matcher = readFile(pom) = ~'<version>(.+)</version>'
    matcher ? matcher[0][1] : null
}

def getGroupIdFromPom(pom) {
    def matcher = readFile(pom) = ~'<groupId>(.+)</groupId>'
    matcher ? matcher[0][1] : null
}

def getArtifactIdFromPom(pom) {
    def matcher = readFile(pom) = ~'<artifactId>(.+)</artifactId>'
    matcher ? matcher[0][1] : null
}

def String getBuildVersion(pom) {
    return getVersionFromPom(pom).minus("-SNAPSHOT") + '.' + env.BUILD_NUMBER
}
