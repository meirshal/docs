# Deploying a static website in Trunk Based Development

### Motivation
Building your system once and deploying it to each environment separately with different configuration.

### Problem
For every process you run, extract configuration to environment variables that are manages outside of the code base.
However, when building a static website you don’t run a process but build a static bundle of html/js files that are stored on S3 are delivered though cloudfront. No environment variables exist here.

### Naive solution
Most reactjs fw/build systems (like next.js/CRA, etc.) allow you to build your static bundle. This build can use environment variables that the transpiled code can access as if they were regular env vars (they are just injected to the DOM and read from there). This requires to build to code separately for each environment.

### Better solution
#### Goal
Build the website once and inject configuration to the built bundle on deployment.
#### How to do that
1. Modify the html template (usually under public/index.html) to contain the following tag under the `<head>` tag: 
```
<script>
   window.environment = JSON.parse('{{ ENVIRONMENT_CONFIG }}');
</script>
```
2. Create a new ts that uses this injected configuration, to be used whenever configuration access is needed:
```typescript
const environment = window['environment'] || {};
const getEnv = (key: string, defaultValue?) => environment[key] || process.env[key] || defaultValue;

export const MY_ENV_VAR = getEnv('MY_ENV_VAR');
```
Note that when running locally you can still use environment.

3. When building your project, build it without environment specific environment variables.
4. On deployment, use [Mustache](https://github.com/tests-always-included/mo) to replace `{{ ENVIRONMENT_CONFIG }}` in your built bundle with a JSON containing the configuration:
```shell
environment_config='{ "MY_ENV_VAR": "VALUE" }'
ENVIRONMENT_CONFIG=$environment_config mo build/index.html > build/index.html.dist
mv build/index.html.dist build/index.html
```
