#!/usr/bin/env groovy

node('rhel8') {
    stage('Install requirements') {
        def nodeHome = tool 'nodejs-12.13.1'
        env.PATH = "${env.PATH}:${nodeHome}/bin"
        sh 'node -v'
        sh 'npm -v'
    }

    stage('Download VSIX files') {
        sh 'wget "https://github.com/kiegroup/kie-tooling-store/releases/download/$VERSION/vscode_extension_bpmn_editor_$VERSION.vsix"'
        sh 'wget "https://github.com/kiegroup/kie-tooling-store/releases/download/$VERSION/vscode_extension_dmn_editor_$VERSION.vsix"'
        sh 'wget "https://github.com/kiegroup/kie-tooling-store/releases/download/$VERSION/vscode_extension_pmml_editor_$VERSION.vsix"'
        sh 'wget "https://github.com/kiegroup/kie-tooling-store/releases/download/$VERSION/vscode_extension_red_hat_business_automation_bundle_$VERSION.vsix"'
        sh 'wget "https://github.com/kiegroup/kie-tooling-store/releases/download/$VERSION/vscode_extension_kogito_bundle_$VERSION.vsix"'
        sh 'md5sum *.vsix'
    }

    stage('Archive VSIX files') {
        def vsix = findFiles(glob: '**.vsix')
        archiveArtifacts artifacts: '**.vsix'
    }

    if (publishToMarketPlace.equals('true') || publishToOVSX.equals('true')) {
        timeout(time:1, unit:'DAYS') {
            input message:'Approve deployment?', submitter: 'gcaponet,tfernand'
        }

        def vsix_editors = findFiles(glob: '**editor*.vsix')
        def vsix_redhat_bundle = findFiles(glob: '**red_hat*.vsix')
        def vsix_kogito_bundle = findFiles(glob: '**kogito*.vsix')

        if (publishToMarketPlace.equals('true')) {
            stage('Publish to VS Code Marketplace') {
                sh 'npm install -g vsce'
                withCredentials([[$class: 'StringBinding', credentialsId: 'vscode_java_marketplace', variable: 'TOKEN']]) {
                    sh 'vsce publish -p ${TOKEN} --packagePath' + " ${vsix_editors[0].path}"
                    sh 'vsce publish -p ${TOKEN} --packagePath' + " ${vsix_editors[1].path}"
                    sh 'vsce publish -p ${TOKEN} --packagePath' + " ${vsix_editors[2].path}"
                    sh 'vsce publish -p ${TOKEN} --packagePath' + " ${vsix_redhat_bundle[0].path}"
                }
                withCredentials([[$class: 'StringBinding', credentialsId: 'kie-vscode-token', variable: 'KIE_TOKEN']]) {
                    sh 'vsce publish -p ${KIE_TOKEN} --packagePath' + " ${vsix_kogito_bundle[0].path}"
                }
            }
        }

        if (publishToOVSX.equals('true')) {
            stage('Publish to Open-vsx Marketplace') {
                sh "npm install -g ovsx"
                withCredentials([[$class: 'StringBinding', credentialsId: 'open-vsx-access-token', variable: 'OVSX_TOKEN']]) {
                    sh 'ovsx publish -p ${OVSX_TOKEN}' + " ${vsix_editors[0].path}"
                    sh 'ovsx publish -p ${OVSX_TOKEN}' + " ${vsix_editors[1].path}"
                    sh 'ovsx publish -p ${OVSX_TOKEN}' + " ${vsix_editors[2].path}"
                    sh 'ovsx publish -p ${OVSX_TOKEN}' + " ${vsix_redhat_bundle[0].path}"
                }
                withCredentials([[$class: 'StringBinding', credentialsId: 'kie-openvsx-token', variable: 'KIE_OVSX_TOKEN']]) {
                    sh 'ovsx publish -p ${KIE_OVSX_TOKEN}' + " ${vsix_kogito_bundle[0].path}"
                }
            }
        }
    }
}
