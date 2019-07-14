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
	}
	stage("init") {
		sh '''#!/bin/sh
			sudo apt-get update
			sudo apt-get -y install build-essential scons pkg-config \
				libx11-dev libxcursor-dev libxinerama-dev libgl1-mesa-dev libglu-dev libasound2-dev \
				libpulse-dev libudev-dev libxi-dev libxrandr-dev yasm libfreetype6-dev
			cd godot-updated
			misc/travis/android-tools-linux.sh

		'''
	}
	stage("build-tools-linux") {
		sh '''#!/bin/sh
			cd godot-updated
			scons platform=x11 -j8 tools=yes target=debug
			scons platform=x11 -j8 tools=yes target=release_debug
			scons platform=server -j8 tools=yes target=release_debug
		'''
	}
	stage("build-templates-linux") {
		sh '''#!/bin/sh
			cd godot-updated
			scons platform=x11 -j8 tools=no target=debug
			scons platform=x11 -j8 tools=no target=release_debug
			scons platform=x11 -j8 tools=no target=release
			scons platform=server -j8 tools=no target=release_debug
		'''
	}
	stage("artifacts") {
		sh '''#!/bin/sh
			rm -Rf godot-templates
			mkdir godot-templates
			cp godot-updated/bin/* godot-templates
			tar zcf godot-templates.tar.gz godot-templates
			zip -r godot-templates.zip godot-templates
		'''
		archiveArtifacts artifacts: "godot-templates.tar.gz", onlyIfSuccessful: true
		archiveArtifacts artifacts: "godot-templates.zip", onlyIfSuccessful: true
	}
}
