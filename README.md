# Build and deploy npm package
This action builds and publishes an npm package to GitHub Packages.  
  It is intended for use in a CI/CD pipeline to automate the process of building and publishing a package.  

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
    required: false

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
  ```

  ## Secrets
  ```yml
  GITHUB_TOKEN:
      description: 'GitHub token for authentication'
      required: true

    NPM_TOKEN:
      description: 'NPM token for publishing packages'
      required: true
 ```
