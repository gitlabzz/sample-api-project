node {

    def mvnHome
    def branchName
    def targetEnvironment
    def pullRequest
    def apimHost

    stage('Initialize') {
        branchName = BRANCH_NAME
        echo "checking if it's a pull request branch!"
        if (branchName.toUpperCase().startsWith("PR-")) {
            echo "found pull request '${branchName}', so targetting it to the 'DEV' environment!!!"
            pullRequest = branchName.substring(branchName.lastIndexOf("-") + 1)
            echo "Pull request is   ========================================>  ${pullRequest}."
        }
    }

    stage("Checkout Code (${pullRequest ? 'PR-' + pullRequest : branchName})") {
        if (pullRequest) {
            echo "Checking out pull request  ========================================> ${branchName}"
            try {
                git branch: '${BRANCH_NAME}', credentialsId: '2bc605b8-3d32-4c7b-84e2-4d858bc31c46', url: 'https://github.com/gitlabzz/sample-api-project.git'
            } catch (exception) {
                sh '''
                    git fetch origin +refs/pull/''' + pullRequest + '''/merge
                    git checkout FETCH_HEAD
                '''
                branchName = "dev"
                echo "targeting build for pull request ${pullRequest} to '${branchName}' environment"
            }
            echo "Check out for pull request '${BRANCH_NAME}' is successfully completed!"

        } else {
            echo "Checking out branch  ========================================> ${BRANCH_NAME}"
            git branch: '${BRANCH_NAME}', credentialsId: '2bc605b8-3d32-4c7b-84e2-4d858bc31c46', url: 'https://github.com/gitlabzz/sample-api-project.git'
            echo "Check out for '${BRANCH_NAME}' is successfully completed!"
        }
    }

    stage("Prepare Environment (${branchName})") {
        echo "Preparing build environment for branch ---------------------------------> '${branchName}'"
        targetEnvironment = branchName.toUpperCase()
        mvnHome = tool 'M3'

        echo "Target Environment is ---------------------------------> ${targetEnvironment}"

        def apimHostEnvVar = "APIM_${branchName.toUpperCase()}_HOST"
        echo "Using Global Environment Variable: '${apimHostEnvVar}'"
        apimHost = env.getProperty(apimHostEnvVar)
        echo "The Global Environment Variable: '${apimHostEnvVar}' is set to '${apimHost}'"

        withCredentials([usernamePassword(credentialsId: "APIM_ADMIN_USERNAME_PASSWORD_${targetEnvironment}_ENV", passwordVariable: 'password', usernameVariable: 'username')]) {
            env.APIM_HOST = apimHost
            env.APIM_ADMIN_USER = username
            env.APIM_ADMIN_PASSWORD = password
            env.AXWAY_APIM_CLI_HOME = "src/main/environments/${branchName}"
            env.ENV = branchName

            echo "Setting 'AXWAY_APIM_CLI_HOME' to --------------------------------------> ${AXWAY_APIM_CLI_HOME}"
            echo "Using 'conf/env.properties' file from ---------------------------------> '${AXWAY_APIM_CLI_HOME}'"
            echo "Setting APIM_HOST to --------------------------------------------------> ${apimHost}"
            echo "Getting APIM login credentails from -----------------------------------> 'APIM_ADMIN_USERNAME_PASSWORD_${targetEnvironment}_ENV' secret"
            echo "Setting APIM_ADMIN_USER to --------------------------------------------> ${username}"
            echo "Setting APIM_ADMIN_PASSWORD from secret 'APIM_ADMIN_USERNAME_PASSWORD_${targetEnvironment}_ENV'"
            echo "Setting EVN -----------------------------------------------------------> ${branchName}"
        }
    }

    stage('Prepare Package') {
        echo "Preparing '.zip' file using 'mvn install'"
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" clean install'
            }
        }
        echo "Prepared'.zip' file successfully"
    }

    stage("APIs Before Publish (${branchName})") {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                echo "Retrieving published APIs from '${branchName}' before publish"
                env.MAVEN_OPTS = '-Xms256m -Xmx512m -Dlog4j.configurationFile=src/main/resources/log4j/log4j2.xml'
                echo "Setting MAVEN_OPTS environment variable for maven build to '${env.MAVEN_OPTS}'"
                echo "------------------------------------ EXISTING APIs BEFORE PUBLISH ------------------------------------"
                sh '"$MVN_HOME/bin/mvn" exec:java@list-api'
            }
        }
    }

    stage("Publish API (${branchName})") {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                echo "Publishing to environment: '${branchName}'"
                env.MAVEN_OPTS = '-Xms256m -Xmx512m -Dlog4j.configurationFile=src/main/resources/log4j/log4j2.xml'
                sh '"$MVN_HOME/bin/mvn" exec:java@import-api'
            }
        }
    }

    stage("APIs After Publish (${branchName})") {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                echo "Retrieving APIs from '${branchName}' after publish"
                env.MAVEN_OPTS = '-Xms256m -Xmx512m -Dlog4j.configurationFile=src/main/resources/log4j/log4j2.xml'
                echo "------------------------------------ EXISTING APIs AFTER PUBLISH ------------------------------------"
                sh '"$MVN_HOME/bin/mvn" exec:java@list-api'
            }
        }
    }

    stage('Push Package to Nexus') {
        echo "Pushing .Zip bundle to Nexus repository"
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                // TODO: later make it deploy instead of install
                sh '"$MVN_HOME/bin/mvn" clean install'
            }
        }
    }
}