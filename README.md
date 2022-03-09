### command line invoke maven

#### Enable Debug logs for apim-cli

`export MAVEN_OPTS='-Xms64m -Xmx256m -Dlog4j.configurationFile=src/main/resources/log4j/log4j2.xml'`

#### Folder containing 'DEV' API Manager connectivity setup via environment variables passed from command line or Jenkins pipeline

- APIM_HOST
- APIM_ADMIN_USER
- APIM_ADMIN_PASSWORD

export AXWAY_APIM_CLI_HOME=src/main/environments/dev

#### Commandline setup, set variables for build

- `export APIM_HOST=dev.apim.demo.bct`
- `export APIM_ADMIN_PASSWORD=changeme`
- `export APIM_ADMIN_USER=apiadmin`

#### Import into API Manager

`mvn clean install exec:java@import-api`

#### List published APIs

`mvn clean install exec:java@list-api`