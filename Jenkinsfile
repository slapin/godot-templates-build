def git_clone(url, branch, dirname)
{
	checkout([$class: 'GitSCM',
	branches: [[name: "*/${branch}"]],
	doGenerateSubmoduleConfigurations: false,
	extensions: [[$class: 'LocalBranch', localBranch: branch],
		    [$class: 'RelativeTargetDirectory',
	relativeTargetDir: dirname]],
	submoduleCfg: [],
	userRemoteConfigs: [[url: url]]])
}
node('docker && ubuntu-16.04') {
	stage("clone") {
		git_clone('git://github.com/slapin/godot', 'navigation', 'godot-updated')
	sh '''#!/bin/sh
		sudo apt-get install scons
	'''
	sh '''#!/bin/sh
		cd godot-updated
		scons platform=x11 tools=no
	'''
	sh '''#!/bin/sh
		rm -Rf godot-templates
		mkdir godot-templates
		cp godot-updated/bin/* godot-templates
		tar zcf godot-templates.tar.gz godot-templates
		zip -r godot-templates.zip godot-templates
	'''
	archiveArtifacts artifacts: godot-templates.tar.gz onlyIfSuccessful: true
	archiveArtifacts artifacts: godot-templates.zip onlyIfSuccessful: true
}
