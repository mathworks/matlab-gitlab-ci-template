# Use MATLAB with GitLab CI/CD
You can use [GitLab&trade; CI/CD](https://docs.gitlab.com/ee/ci/index.html) to run MATLAB&reg; scripts, functions, and statements as part of your pipeline. You also can run your MATLAB and Simulink&reg; tests and generate artifacts such as JUnit test results and Cobertura code coverage reports. 

The `MATLAB.gitlab-ci.yml` [template](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/MATLAB.gitlab-ci.yml) provides you with jobs that show how to run MATLAB scripts, functions, statements, and tests. To run MATLAB code and Simulink models based on this template, you must use the Docker executor to run MATLAB within a container. The [MATLAB Container on Docker Hub](https://www.mathworks.com/help/cloudcenter/ug/matlab-container-on-docker-hub.html) lets you run your build using MATLAB R2020b or a later release. If your build requires additional toolboxes, use a custom MATLAB container instead. For more information on how to create and use a custom MATLAB container, see [Create a Custom MATLAB Container](https://www.mathworks.com/help/cloudcenter/ug/create-a-custom-matlab-container.html).

>**Note:** In addition to the Docker executor, GitLab Runner implements other types of executors that can be used to run your builds. See [Executors](https://docs.gitlab.com/runner/executors/) for more information.

## MATLAB.gitlab-ci.yml Template
You can access the `MATLAB.gitlab-ci.yml` template when you create a `.gitlab-ci.yml` file in the UI.

![template](https://user-images.githubusercontent.com/48831250/166474348-2e106005-23eb-4d62-a0ba-3387bbfcb20a.png)

The template has three jobs:

* `command` — Run MATLAB scripts, functions, and statements.                
* `test` — Run tests authored using the MATLAB unit testing framework or Simulink Test&trade;.
* `test_artifacts` — Run MATLAB and Simulink tests, and generate test and coverage artifacts.

The jobs in the template use the `matlab -batch` syntax to start MATLAB. Additionally, they incorporate the contents of a hidden `.matlab_defaults` job. You need to configure this job before running the `command`, `test`, and `test_artifacts` jobs. To configure the job:

* Specify the name of the MATLAB container image you want to use.
* Set the `MLM_LICENSE_FILE` environment variable using the port number and DNS address for your network license manager.

```yaml
.matlab_defaults:
  image:
    name: mathworks/matlab:latest  # Replace the value with the name of the MATLAB container image you want to use
    entrypoint: [""]
  variables:
    MLM_LICENSE_FILE: 27000@MyLicenseServer  # Replace the value with the port number and DNS address for your network license manager

```

## Examples
Each of these examples shows how to specify the contents of a job in your `.gitlab-ci.yml`.

### Run a Script
Run the commands in a file named `myscript.m` in the root of your repository.

```yaml
command:
   extends: .matlab_defaults
   script: matlab -batch "myscript"
```
### Run Tests
Run the tests in your [MATLAB project](https://www.mathworks.com/help/matlab/projects.html) and fail the build if any of the tests fail.

```yaml
test:
  extends: .matlab_defaults
  script: matlab -batch "results = runtests('IncludeSubfolders',true), assertSuccess(results);"
```
### Run Tests and Generate Artifacts
Run the tests in your MATLAB project, and generate a JUnit test results report and a Cobertura code coverage report. Fail the build if any of the tests fail.

```yaml
test_artifacts:
  extends: .matlab_defaults
  script: |
    cat <<- 'BLOCK' > runAllTests.m
          import matlab.unittest.TestRunner
          import matlab.unittest.Verbosity
          import matlab.unittest.plugins.CodeCoveragePlugin
          import matlab.unittest.plugins.XMLPlugin
          import matlab.unittest.plugins.codecoverage.CoberturaFormat
          suite = testsuite(pwd,'IncludeSubfolders',true);
          [~,~] = mkdir('artifacts')
          runner = TestRunner.withTextOutput('OutputDetail',Verbosity.Detailed);
          runner.addPlugin(XMLPlugin.producingJUnitFormat('artifacts/results.xml'))
          % Replace `pwd` with the location of the folder containing source code
          runner.addPlugin(CodeCoveragePlugin.forFolder(pwd,'IncludingSubfolders',true, ...
          'Producing',CoberturaFormat('artifacts/cobertura.xml')))
          results = runner.run(suite)
          assertSuccess(results);
    BLOCK
    matlab -batch runAllTests
artifacts:
    reports:
      junit: "./artifacts/results.xml"
      coverage_report:
        coverage_format: cobertura
        path: "./artifacts/cobertura.xml"
    paths:
      - "./artifacts"
```

## See Also
- [Continuous Integration with MATLAB and Simulink](https://www.mathworks.com/solutions/continuous-integration.html)
- [Continuous Integration for Verification of Simulink Models Using GitLab](https://www.mathworks.com/company/newsletters/articles/continuous-integration-for-verification-of-simulink-models-using-gitlab.html)

## Contact Us
If you have any questions or suggestions, please contact MathWorks&reg; at [continuous-integration@mathworks.com](mailto:continuous-integration@mathworks.com).
