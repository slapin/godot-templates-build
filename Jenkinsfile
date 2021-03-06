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
	stage("clean") {
		sh '''#!/bin/sh
			rm -Rf godot-updated mingw-build
		'''
	}
	stage("clone") {
		checkout scm;
		git_clone('git://github.com/slapin/godot', 'navigation', 'godot-updated')
	}
	stage("init") {
		sh '''#!/bin/sh
			sudo apt-get update
			sudo apt-get -y install build-essential scons pkg-config \
				libx11-dev libxcursor-dev libxinerama-dev libgl1-mesa-dev libglu-dev libasound2-dev \
				libpulse-dev libudev-dev libxi-dev libxrandr-dev yasm libfreetype6-dev \
				subversion openjdk-8-jdk
			cd godot-updated
			misc/travis/android-tools-linux.sh
			cd ..
			mkdir butler
			cd butler
			curl -L -o butler.zip https://broth.itch.ovh/butler/linux-amd64/LATEST/archive/default
			unzip butler.zip
			chmod +x butler
			./butler -V
			cd ..
			git clone https://github.com/emscripten-core/emsdk.git
			cd emsdk
			./emsdk install latest
			./emsdk activate latest
			cd ..
			# wget https://netix.dl.sourceforge.net/project/mingw-w64/mingw-w64/mingw-w64-release/mingw-w64-v5.0.4.tar.bz2
			# tar xf mingw-w64-v5.0.4.tar.bz2
			# wget -Ointsafe.h https://raw.githubusercontent.com/Alexpux/mingw-w64/master/mingw-w64-headers/include/intsafe.h
			# cp intsafe.h godot-updated/thirdparty/mbedtls/include/
			#wget https://github.com/slapin/build-mingw/releases/download/2019_29_0715_0205/mingw-build.tar.bz2
			#tar xf mingw-build.tar.bz2

		'''
	}
	stage("build-templates-android") {
		sh '''#!/bin/sh
			cd godot-updated
			export ANDROID_HOME=$(pwd)/godot-dev/build-tools/android-sdk;
			export ANDROID_NDK_ROOT=$(pwd)/godot-dev/build-tools/android-ndk;

			set -e
			scons verbose=yes progress=no platform=android -j16 tools=no target=release_debug android_arch=armv7
			scons verbose=yes progress=no platform=android -j16 tools=no target=release_debug android_arch=arm64v8
			scons verbose=yes progress=no platform=android -j16 tools=no target=release_debug android_arch=x86
			scons verbose=yes progress=no platform=android -j16 tools=no target=release android_arch=armv7
			scons verbose=yes progress=no platform=android -j16 tools=no target=release android_arch=arm64v8
			scons verbose=yes progress=no platform=android -j16 tools=no target=release android_arch=x86
			cd platform/android/java
			./gradlew build
		'''
		sh '''#!/bin/sh
			find ./godot-updated -name '*.apk' -ls
		'''
	}
	stage("build-tools-linux") {
		sh '''#!/bin/sh
			cd godot-updated
			set -e
			scons platform=x11 -j16 tools=yes target=debug
			scons platform=x11 -j16 tools=yes target=release_debug
			scons platform=server -j16 tools=yes target=debug
			scons platform=server -j16 tools=yes target=release_debug
		'''
	}
	stage("build-templates-linux") {
		sh '''#!/bin/sh
			cd godot-updated
			set -e
			scons verbose=yes progress=no platform=x11 -j16 tools=no target=debug
			scons verbose=yes progress=no platform=x11 -j16 tools=no target=release_debug
			scons verbose=yes progress=no platform=x11 -j16 tools=no target=release
			scons verbose=yes progress=no platform=server -j16 tools=no target=release_debug
		'''
	}
	stage("build-templates-web") {
		sh '''#!/bin/bash
			cd emsdk
			. emsdk_env.sh
			cd ..
			cd godot-updated
			git log |head -20
			scons verbose=yes progress=no platform=javascript -j16 tools=no target=release javascript_eval=no
			scons verbose=yes progress=no platform=javascript -j16 tools=no target=release_debug javascript_eval=no
		'''
	}
	stage("stash") {
		stash includes: 'godot-updated/bin/*', name: 'binaries-main'
		stash includes: 'godot-updated/platform/android/java/app/build/outputs/apk/**/*.apk', name: 'binaries-android'
	}
/*
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
*/
}
node('docker && ubuntu-18.04') {
	stage("clean") {
		sh '''#!/bin/sh
			rm -Rf godot-updated-2
		'''
	}
	stage("clone") {
		checkout scm;
		git_clone('git://github.com/slapin/godot', 'navigation', 'godot-updated-2')
	}
	stage("init") {
		sh '''#!/bin/sh
			sudo apt-get update
			sudo apt-get -y install build-essential scons pkg-config \
				libx11-dev libxcursor-dev libxinerama-dev libgl1-mesa-dev libglu-dev libasound2-dev \
				libpulse-dev libudev-dev libxi-dev libxrandr-dev yasm libfreetype6-dev texinfo \
				texi2html subversion
			# cd godot-updated
			# misc/travis/android-tools-linux.sh
			# cd ..
			# mkdir butler
			# cd butler
			# curl -L -o butler.zip https://broth.itch.ovh/butler/linux-amd64/LATEST/archive/default
			# unzip butler.zip
			# chmod +x butler
			# ./butler -V
			# cd ..
			sudo update-alternatives --set i686-w64-mingw32-gcc /usr/bin/i686-w64-mingw32-gcc-posix
			sudo update-alternatives --set i686-w64-mingw32-g++ /usr/bin/i686-w64-mingw32-g++-posix
			sudo update-alternatives --set x86_64-w64-mingw32-gcc /usr/bin/x86_64-w64-mingw32-gcc-posix
			sudo update-alternatives --set x86_64-w64-mingw32-g++ /usr/bin/x86_64-w64-mingw32-g++-posix
		'''
	}
	stage("build-templates-windows") {
		sh '''#!/bin/sh
			cd godot-updated-2
			x86_64-w64-mingw32-g++ --version
			x86_64-w64-mingw32-g++ -v
			set -e
			scons verbose=yes progress=no platform=windows -j16 tools=no target=debug bits=64
			scons verbose=yes progress=no platform=windows -j16 tools=no target=debug bits=32
			scons verbose=yes progress=no platform=windows -j16 tools=no target=release_debug bits=64
			scons verbose=yes progress=no platform=windows -j16 tools=no target=release_debug bits=32
			scons verbose=yes progress=no platform=windows -j16 tools=no target=release bits=64
			scons verbose=yes progress=no platform=windows -j16 tools=no target=release bits=32
		'''
	}
	stage("build-tools-windows") {
		sh '''#!/bin/sh
			cd godot-updated-2
			x86_64-w64-mingw32-g++ --version
			x86_64-w64-mingw32-g++ -v
			set -e
			scons verbose=yes progress=no platform=windows -j16 tools=yes target=release_debug bits=64
			scons verbose=yes progress=no platform=windows -j16 tools=yes target=release_debug bits=32
		'''
		stash includes: 'godot-updated-2/bin/*', name: 'binaries-windows'
	}
	stage("artifacts") {
		unstash 'binaries-main'
		unstash 'binaries-windows'
		unstash 'binaries-android'
		sh '''#!/bin/sh
			rm -Rf godot-templates
			mkdir godot-templates
			cp godot-updated/bin/* godot-templates
			cp godot-updated/platform/android/java/app/build/outputs/apk/debug/android_debug.apk godot-templates/android_debug.apk
			cp platform/android/java/app/build/outputs/apk/release/android_release.apk godot-templates/android_release.apk
			cp godot-updated/platform/android/java/app/build/outputs/apk/debug/java-debug-unsigned.apk godot-templates/android_debug.apk
			cp godot-updated/platform/android/java/app/build/outputs/apk/release/java-release-unsigned.apk godot-templates/android_release.apk
			cp godot-updated-2/bin/* godot-templates
			tar zcf godot-templates.tar.gz godot-templates
			zip -r godot-templates.zip godot-templates
		'''
		archiveArtifacts artifacts: "godot-templates.tar.gz", onlyIfSuccessful: true
		archiveArtifacts artifacts: "godot-templates.zip", onlyIfSuccessful: true
		withCredentials([string(credentialsId: 'github-token', variable: 'gh_token')]) {
			withEnv(["TOKEN=$gh_token"]) {
				sh '''#!/bin/sh
					./upload-github-release-asset.sh github_api_token=$TOKEN \
						owner=slapin repo=godot-templates-build \
						tag=$(date +%Y_%V_%m%d_%H%M) filename=./godot-templates.tar.gz
				'''
			}
		}
	}
}

