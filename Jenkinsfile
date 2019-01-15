node ('linux') {
    timeout(60) {
        stage ('Checkout') {
            checkout scm
        }

        stage ('Build') {
            String jdk = '8'
            String jdkTool = "jdk${jdk}"
            List<String> env = [
                    "JAVA_HOME=${tool jdkTool}",
                    'PATH+JAVA=${JAVA_HOME}/bin',
            ]
            String command
            List<String> mavenOptions = [
                    '--batch-mode',
                    '--errors',
                    '--update-snapshots',
                    '-Dmaven.test.failure.ignore',
            ]
            if (jdk.toInteger() > 7 && infra.isRunningOnJenkinsInfra()) {
                /* Azure mirror only works for sufficiently new versions of the JDK due to Letsencrypt cert */
                def settingsXml = "${pwd tmp: true}/settings-azure.xml"
                writeFile file: settingsXml, text: libraryResource('settings-azure.xml')
                mavenOptions += "-s $settingsXml"
            }
            mavenOptions += "clean verify jacoco:prepare-agent test jacoco:report"
            command = "mvn ${mavenOptions.join(' ')}"
            env << "PATH+MAVEN=${tool 'mvn'}/bin"

            withEnv(env) {
                sh command
            }

            junit testResults: '**/target/*-reports/TEST-*.xml'
            recordIssues enabledForFailure: true, tool: mavenConsole()
            recordIssues enabledForFailure: true, tools: [java(), javaDoc()], sourceCodeEncoding: 'UTF-8'
            recordIssues enabledForFailure: true, tool: errorProne(), sourceCodeEncoding: 'UTF-8'
            recordIssues enabledForFailure: true, tool: checkStyle(), sourceCodeEncoding: 'UTF-8'
            recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml'), sourceCodeEncoding: 'UTF-8'
            recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml'), sourceCodeEncoding: 'UTF-8'
            recordIssues enabledForFailure: true, tool: spotBugs(), sourceCodeEncoding: 'UTF-8'
            recordIssues enabledForFailure: true, tool: taskScanner(includePattern:'**/*.java', excludePattern:'target/**/*', highTags:'FIXME', normalTags:'TODO'), sourceCodeEncoding: 'UTF-8'
            jacoco()
        }
    }
}

node ('windows') {
    timeout(60) {
        stage ('Checkout') {
            checkout scm
        }

        stage ('Build') {
            String jdk = '8'
            String jdkTool = "jdk${jdk}"
            List<String> env = [
                    "JAVA_HOME=${tool jdkTool}",
                    'PATH+JAVA=${JAVA_HOME}/bin',
            ]
            String command
            List<String> mavenOptions = [
                    '--batch-mode',
                    '--errors',
                    '--update-snapshots',
                    '-Dmaven.test.failure.ignore',
            ]
            if (jdk.toInteger() > 7 && infra.isRunningOnJenkinsInfra()) {
                /* Azure mirror only works for sufficiently new versions of the JDK due to Letsencrypt cert */
                def settingsXml = "${pwd tmp: true}/settings-azure.xml"
                writeFile file: settingsXml, text: libraryResource('settings-azure.xml')
                mavenOptions += "-s $settingsXml"
            }
            mavenOptions += "clean verify"
            command = "mvn ${mavenOptions.join(' ')}"
            env << "PATH+MAVEN=${tool 'mvn'}/bin"

            withEnv(env) {
                bat command
            }

            junit testResults: '**/target/*-reports/TEST-*.xml'
        }
    }
}