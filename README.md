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

deployment_branch_name:
    description: 'Branch name for deployment'
    required: false
    default: 'main'

version_command:
    description: 'Command to create a version of the package'
    required: true

publish_command:
    description: 'Command to publish the package'
    required: true

is_pre_release:
    description: 'Indicates if the package is a pre-release'
    required: false
    type: boolean
    default: false

pre_release_branch_name:
    description: 'Branch name for pre-release deployment'
    required: false
    type: string
    default: 'dev'
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
