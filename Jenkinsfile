DOTNET_PATH = '%dotnet%' 
MSBUILD = '%msbuild%'

pipeline {
    parameters {
        choice(name: 'build', choices: ['Debug', 'Release'], description: "Release or Debug.")
        choice(name: 'release', choices: ['alpha', 'beta', 'release'], description: "alpha - risky builds that may crash, beta - more mature builds testing before release release - builds ready for deployment to user")
		string(name: 'version', defaultValue: '1.0.0', description: 'Application Version')
		
		// After build step there is a copy step, where we copy the build to the CloudXR servers
		string(name: 'servers', defaultValue: 'server1', description: 'Separate server name by space')
		booleanParam(name: 'copy', defaultValue: false, description: 'whether to copy the builds to the VM drives. Skip this step when you are not on network')
    }

    agent any 
    
    environment {
        appname = "ConsoleApp"
        release_name = "${ "${release}" == "alpha" || "${release}" == "beta" ? "${release}" : "" }" 
        target = "${ "${build}" == "Release" ? "${appname}${release_name}.exe" : " ${appname}_Debug_${release_name}.exe" }" // append debug for debug builds, nothing for release builds
    }

    stages {
        stage ('Build') { steps { script { bat """
           
        ::\"${DOTNET_PATH}\" restore
    	::\"${DOTNET_PATH}\" rebuild
        ::\"${DOTNET_PATH}\" build --configuration ${build}
        \"${MSBUILD}\" \"${appname}.sln\" -p:Configuration=${build} 	
		
		\"${DOTNET_PATH}\" \"${scannerhome}\\SonarScanner.MSBuild.dll\" begin /key:${appname} /d:sonar.login=${login_token}
		\"${DOTNET_PATH}\" build \"${appname}.sln\"
		\"${DOTNET_PATH}\" \"${scannerhome}\\SonarScanner.MSBuild.dll\" end /d:sonar.login=${login_token}		

        """ }}} // end build stage

        stage ('Copy') { 
        when { environment name: 'copy', value: 'true' }
        steps { script { bat """
        

        (for %%s in (%servers%) do (

				echo %%s
				rmdir \\\\%%s\\Build\\${appname} /s /q
				echo d|xcopy /y /e /s bin\\${build} \\\\%%s\\Build\\${appname}
				echo.>version.txt
				echo ${version} >>version.txt
				xcopy /y version.txt \\\\%%s\\Build\\${appname}
			))
        """ }}} // end copy stage


    } // end stages
}
