# Use MATLAB with GitLab CI/CD
You can use [GitLab&trade; CI/CD](https://docs.gitlab.com/ee/ci/index.html) to run MATLAB&reg; scripts, functions, and statements as part of your pipeline. You also can run your MATLAB and Simulink&reg; tests and generate artifacts such as JUnit test results and Cobertura code coverage reports. 

To run MATLAB code and Simulink models with GitLab CI/CD, your runner must have MATLAB (R2019a or later) installed. If you do not have a runner, install [GitLab Runner](https://docs.gitlab.com/runner/install/) and register your computer as a runner for your instance, project, or group.

The runner uses the topmost MATLAB version on the system path. It fails the build if MATLAB is not on the path.  

## MATLAB.gitlab-ci.yml Template
The `MATLAB.gitlab-ci.yml` [template](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/MATLAB.gitlab-ci.yml) provides you with jobs that show how to run MATLAB scripts, functions, statements, and tests. You can choose this template when you create a `.gitlab-ci.yml` file in the UI.

![template](https://user-images.githubusercontent.com/48831250/166474348-2e106005-23eb-4d62-a0ba-3387bbfcb20a.png)

The template has three jobs:

* `command`: Run MATLAB scripts, functions, and statements.                
* `test`: Run tests authored using the MATLAB unit testing framework or Simulink Test&trade;.
* `test_artifacts`: Run MATLAB and Simulink tests, and generate test and coverage artifacts.

## Examples
Each of these examples shows how to specify the contents of your `.gitlab-ci.yml` file by using one of the jobs in the template.

### Run a Script
Run the commands in a file named `myscript.m` in the root of your repository.

```yaml
command:
   script: matlab -batch "myscript"
```
### Run Tests
Run the tests in your [MATLAB project](https://www.mathworks.com/help/matlab/projects.html) and fail the build if any of the tests fail.

```yaml
test:
  script: matlab -batch "results = runtests('IncludeSubfolders',true), assertSuccess(results);"
```
### Run Tests and Generate Artifacts
Run the tests in your MATLAB project, and generate a JUnit test results report and a Cobertura code coverage report. Fail the build if any of the tests fail.

```yaml
tests_artifacts:
  script: |
       matlab -batch "
       import matlab.unittest.TestRunner
       import matlab.unittest.Verbosity
       import matlab.unittest.plugins.CodeCoveragePlugin
       import matlab.unittest.plugins.XMLPlugin
       import matlab.unittest.plugins.codecoverage.CoberturaFormat
   
       suite = testsuite(pwd,'IncludeSubfolders',true);
   
       [~,~] = mkdir('artifacts');
   
       runner = TestRunner.withTextOutput('OutputDetail',Verbosity.Detailed);
       runner.addPlugin(XMLPlugin.producingJUnitFormat('artifacts/results.xml'))
       runner.addPlugin(CodeCoveragePlugin.forFolder(pwd,'IncludingSubfolders',true, ...
           'Producing',CoberturaFormat('artifacts/cobertura.xml')))
   
       results = runner.run(suite)
       assertSuccess(results);"
        
  artifacts:
    reports:
      junit: "./artifacts/results.xml"
      cobertura: "./artifacts/cobertura.xml"
    paths:
    - "./artifacts"
```

## See Also
- [Continuous Integration with MATLAB and Simulink](https://www.mathworks.com/solutions/continuous-integration.html)
- [Continuous Integration for Verification of Simulink Models Using GitLab](https://www.mathworks.com/company/newsletters/articles/continuous-integration-for-verification-of-simulink-models-using-gitlab.html)

## Contact Us
If you have any questions or suggestions, please contact MathWorks&reg; at [continuous-integration@mathworks.com](mailto:continuous-integration@mathworks.com).
