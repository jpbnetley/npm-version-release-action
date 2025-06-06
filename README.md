# Build and deploy npm package
This action builds and publishes an npm package.  
  It is intended for use in a CI/CD pipeline to automate the process of building and publishing a package.  

## Notes
- For the gh action to create pull requests, set the permission for the repo to "can create and merge pull requests"
- for the action that creates the pull request, ensure the pull request write permission is granted
```yml
pull-requests: write
```

- For publishing the package, the action required
```yml
contents: write
packages: write
```

## Order of events
In order to maintain pre-release and production release versions, the order which the releases done are important.  
Development continues as always on the dev branch.

Once a release needs to be cut, a pull request is made to the main branch.  
Once merged, a build can be kicked off by running the `release-main` workflow.
This workflow will ask the branch to run on, and the release version.  
The release version uses [semantic versioning](https://semver.org/) to determine the release version.  
Thus the patch, minor, major options are provided when doing the production release.

Once the production release is completed, a pull request is made to merge the new version change back down to dev.

*NOTE*
It is important to merge the production release version changes down to dev before any new changes on dev are made.  
This ensures that the correct version is used on dev for the next dev releases.

## Required permissions
  ```yml
contents: write
packages: write
pull-requests: write
```

  ## Inputs
  ```yml
node_version:
    description: 'Node.js version to use for the build'
    required: false
    default: '18.x'

package_manager:
    description: 'Package manager to use (npm or yarn)'
    required: false
    default: 'npm'

path_to_package_json:
    description: 'Path to package.json file'
    required: false
    default: './package.json'

path_to_package_lock:
    description: 'Path to package-lock.json or yarn.lock file'
    required: false
    default: 'package-lock.json'

registry_url:
    description: 'URL of the npm registry to publish to'
    required: true

install_command:
    description: 'Command to install dependencies'
    required: false
    default: 'npm ci'

build_command:
    description: 'Command to build the package'
    required: false
    default: 'npm run build'

build_output_path:
    description: 'Path to the build output directory'
    required: false
    default: 'dist'

version_command:
    description: 'Command to create a version of the package'
    required: true

publish_command:
    description: 'Command to publish the package'
    required: true

pre_release_branch_name:
    description: 'Branch name for pre-release deployment'
    required: false
    type: string
    default: 'dev'

release_branch_name:
    description: 'Branch name for release deployment'
    required: false
    type: string
    default: 'main'
  ```

## Secrets
  ```yml
  GITHUB_TOKEN:
      description: 'GitHub token for authentication'
      required: true

  NPM_TOKEN:
    description: 'NPM token for publishing packages'
    required: true
    
  NODE_AUTH_TOKEN:
    description: 'Node authentication token for npm registry'
    required: true
 ```

## Example workflow

### Package json of an project
```json
{
  "name": "@jpbnetley/npm-version-release",
  "version": "v1.1.12-dev.0",
  "main": "dist/index.js",
  "module": "dist/index.js",
  "type": "module",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  },
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "scripts": {
    "test": "tsc --noEmit",
    "build": "tsc",
    "version:dev": "npm version --preid=dev prerelease",
    "version:patch": "npm version patch",
    "version:minor": "npm version minor",
    "version:major": "npm version major",
    "version:publish": "npm publish --access public && git push --follow-tags",
    "version:publish:dev": "npm publish --access public --tag dev && git push --follow-tags"
  },
  "author": "Jonathan Netley",
  "license": "ISC",
  "description": "",
  "devDependencies": {
    "typescript": "5.8.3"
  },
  "exports": {
    ".": {
      "import": "./dist/index.js"
    }
  }
}

```
### Github actions for the release
- prerelease (in this case, to a dev branch)

```yml
name: Build and deploy dev

on:
  push:
    branches: 
      - dev

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}

jobs:
  build-and-publish:
    name: Build and publish dev
    permissions:
      contents: write
      packages: write
      pull-requests: write
    uses: jpbnetley/npm-version-release-action/.github/workflows/action.yml@main
    with:
      node_version: 22
      package_manager: 'npm'
      path_to_package_json: './package.json'
      path_to_package_lock: './package-lock.json'
      registry_url: 'https://npm.pkg.github.com'
      install_command: 'npm ci'
      build_command: 'npm run build'
      build_output_path: 'dist'
      version_command: 'npm run version:dev'
      publish_command: 'npm run version:publish:dev'
      pre_release_branch_name: 'dev'
      release_branch_name: 'main'
    secrets: inherit

```

- Production release (in this case to a dev branch)
```yml
name: Build and deploy production
on:
  workflow_dispatch:
   inputs:
      version:
        type: choice
        description: Version type
        options: 
        - patch
        - minor
        - major
      
env:
  node_version: 22
  deployment_branch: main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}

jobs:
  build-and-publish:
    name: Build and publish main
    permissions:
      contents: write
      packages: write
      pull-requests: write
    uses: jpbnetley/npm-version-release-action/.github/workflows/action.yml@main
    with:
      node_version: 22
      package_manager: 'npm'
      path_to_package_json: './package.json'
      path_to_package_lock: './package-lock.json'
      registry_url: 'https://npm.pkg.github.com'
      install_command: 'npm ci'
      build_command: 'npm run build'
      build_output_path: 'dist'
      version_command: 'npm run version:${{ github.event.inputs.version }}'
      publish_command: 'npm run version:publish'
      pre_release_branch_name: 'dev'
      release_branch_name: 'main'
    secrets: inherit

```