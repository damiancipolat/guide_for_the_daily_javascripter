<img src="https://github.com/damcipolat-recargapay/js_developer_guide/blob/master/img/node_logo.jpg?raw=true" width="300px" align="right" />

# Node.js best practices
This document is a summary of good programming practices in js in general.

## Npm
Some interesting tips and commands to use in npm.
 
 #### `npm init`
 Execute this command whenever you start a project from scratch
 
 #### `npm install {dependency} --save`
 Execute this command using the save parameter, when you need to install a new module, the save parameter record the dependecy in the package.json
 
 #### `npm install {dependency} --save--dev`
 Install a new dependency but only for development purposes, example unit testing.
 
 #### `npm install`
 Will install both "dependencies" and "devDependencies" from package.json.
 
 #### `npm install --dev`
 Run this command when you need to install only dev dependencys example into a ci/cd step to run test. Will only install "devDependencies" from package.json
 
 #### `npm install --production`
 Will only install "dependencies" from package.json.
 
 #### `npm audit`
 This command list all the security vulnerabilities of the dependencys installed in the package.json
 
 #### `npm audit --fix`
 Subcommand to automatically install compatible updates to vulnerable dependencies.

## package.json

 - **VERSION**:
 
 Use the `version` attribute to save the current project version follow the SEMVER rules, http://semver.org
 
  ```json
   {
    "name": "api",
    "version": "1.0.0",
    "description": "orders api",
    "main": ""
    }
  ```
  
  > SEMVER rules:
  **MAJOR**: version when you make incompatible API changes.
  **MINOR**: version when you add functionality in a backwards-compatible manner.
  **PATCH**: version when you make backwards-compatible bug fixes.


 - **DEPENDENCIES:**
 
 Make sure you are saving the dependencies modules in the **"devDependencies"** section.
 
 - **SCRIPTS:**
 
Its important to complete the script section of the package.json, the basic script should be:

 ```sh
npm start
npm test
npm deploy
 ```
 
## Node.js
 
- Use npm mayor / npm minor things change the versions.
- Set NODENV in production.
- Split common code in modules.
- Don't use sync function for i/o;
- Use streams.
- Communicate error using exceptions.
- Try/catch if you can solve the exception.
- Good use of promises.
- Promise.all to paralellize always 
- Wrap promise.all to avoid partial executions.
- Async / await instead of promise.then
- Don't use callbacksm replace them by Promise.
- Avoid heavy nesting.
- Avoid use else if.
- Avoid nested if.
- Avoid use global variables.
- Don't abuse installing modules
- Think first in use node core modules instead of search npm modules.
- Create a logger wrapper instead of use console.log, (winston)
- Create private npm modules for general purposes code used in different projects. Reutilizations.
- Avoid the "core" patterns', the idea is avoid putting the entire business code of the application in a set of npm modules
- Don't use class is better to focus in modules that export functions.
- Externalize your config, make easy the test later.

**Stacks**:
Some recomendatios of modules.

- For testing: Jest or Mocha / Chai / Proxyquire.
- For logging: Winston.
- For Api: Expressjs, Hapijs, Restify.
- For SQL: Sequlize.
- For Mongodb: Mongoose.
- For Serverless: Serverless framework or AWS-CDK https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html
- For request: node-fetch or axios.
- For time: moment / moment-timezone.
- For lint: es-lint.
- For schema validation: Joi
