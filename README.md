# SonarQube test

This repositor is work in progress. The aim is to show how to use the SonarQube code checker to check a csolution project in a GitHub workflow.
This project was created from the "Arm-Examples/AVH_CI_Template", so we only need to edit the existing workflow files.

## Things done
1) sing-up for the [free tier SonarQube Cloud account](https://www.sonarsource.com/products/sonarcloud/signup-free/) with the GitHub credentials. This should bring you to the SonarCloud web interface.
2) In the SonarCloud web interface click the + and select "Analyze new project".
3) On the next page one can click "Import an organization from GitHub", which registers the SonarQube GitHub app.
4) This takes you back to SonarCloud, where you "Continue with free plan"
5) Then click "Analyze new project" and you should be able to select one of your repos.
6) When done, click "Set Up" on the right
7) Choose the "new code" setting can continue.
8) On the "Choose your Analysis Method" page, click on "With GitHub Actions".
9) Follow the instrunction on how to create the GitHub Secret with your "SONAR_TOKEN" under "Repository secrets".
10) Create a new file in the project root called "sonar-project.properties". It can contain:

    10.a) Replace the dots with your SonarQube project name, found in SonarCloud.

        sonar.projectKey=...

    10.b) Replace the dots with your SonarQube organization name, also found in SonarCloud.

        sonar.organization=...

    10.c) SonarQube scans by default all files in the folder, so also downloaded vcpkg artefacts and pack files. With this, the scan is limited to the Project folder, where for this project the source code is found.

        sonar.sources=Project/

    10.d) Normally, the SonarQube scanner will not make the GitHub action fail if the Quality Gate requirements are not met. With this setting, it can be made to fail. 

        sonar.qualitygate.wait=true

    10.e) As for the free tier account the default Quality Gate also requires Code Coverage to be 80%, I faked this by running the project in the uVision debugger to create gcov files there and put them in this folder. So the scan can succeed.

        sonar.cfamily.gcov.reportsPath=coverage/

    10.f) The SonarQube includes by default all known source file formats to the Code Coverage value. In this case, there is a python module in the source folder. With that, this python module is excluded from the Code Coverage.

        sonar.coverage.exclusions= **/report.py

    10.g) This is a method to specify an include file to be used on every check of C source files. This can be used for Complier's predefined values.

        sonar.cxx.forceIncludes=predefined_macros.h

11) Things to add in the workflow file's "jobs:" section:

    11.a) To create the Complier's predefined values, the compiler can be used like this:

          - name: generate predefined macros include file
            run: |
              touch predefined_macros.c
              armclang --target=arm-arm-none-eabi -dM -E predefined_macros.c 1>predefined_macros.h 2>/dev/null

    11.b) cbuild (setup) will generate a compile_commands.json file:

          - name: Generate compile_commands.json
            run: |
              echo "Generate compile_commands.json ..."
              cbuild setup get_started.csolution.yml --packs --update-rte --context .debug+avh

    11.c) Do the SonarQube scan:

          - name: SonarQube Scan
            uses: sonarsource/sonarqube-scan-action@v6
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            with:
              args: >
                --define sonar.cfamily.compile-commands=tmp/Project/avh/debug/compile_commands.json

12) When running a workflow this, the SonarQube scan is done and also makes the action fail, if it finds problems.
