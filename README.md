This project demonstrates an issue with the AWS Code Artifact whereby AWS CodeArtifact appears to be stripping `peerDependenciesMeta` info from the `package-lock.json`. Our team has been running into issue with the AWS CDK using the `@aws-cdk/assert` module where it AWALYS attempts to install the optional [Canvas](https://github.com/Automattic/node-canvas) dependency from [JSDom](https://github.com/jsdom/jsdom), which attempts to install native tooling via `node-gyp` and it consistently fails. 

What we have observed is that we get different results when installing directly from `registry.npmjs.org` vs using a mirror defined in AWS CodeArtifact. Using a project that ONLY installs JSDom, it will succeed when using `registry.npmjs.org` and running `npm ls --depth 6` will not install the optional dependency: 

```
➜  npmregistry git:(main) ✗ npm ls --depth 6
npmregistry@1.0.0 /Users/ryan/Documents/Projects/FastTrack/jsdom-npm-test/npmregistry
└─┬ jsdom@16.7.0
  ├── abab@2.0.5
  ├─┬ acorn-globals@6.0.0
  │ ├── acorn-walk@7.2.0
  │ └── acorn@7.4.1
  ├── acorn@8.4.1
  ├── UNMET OPTIONAL DEPENDENCY canvas@^2.5.0
```

However, when running with the `registry` set to a CodeArtifact URL, the following happens:

```
➜  codeartifact git:(main) ✗ npm install
npm ERR! code 1
npm ERR! path /Users/username/Documents/Projects/FastTrack/jsdom-npm-test/codeartifact/node_modules/canvas
npm ERR! command failed
npm ERR! command sh -c node-gyp rebuild
npm ERR! gyp info it worked if it ends with ok
npm ERR! gyp info using node-gyp@7.1.2
npm ERR! gyp info using node@16.5.0 | darwin | x64
npm ERR! gyp info find Python using Python version 3.8.2 found at "/Library/Developer/CommandLineTools/usr/bin/python3"
npm ERR! (node:45294) [DEP0150] DeprecationWarning: Setting process.config is deprecated. In the future the property will be read-only.
npm ERR! (Use `node --trace-deprecation ...` to show where the warning was created)
npm ERR! gyp info spawn /Library/Developer/CommandLineTools/usr/bin/python3
```
We experience this behavior on both macOS and Linux systems

## Steps to reproduce:

Run the NPM Registry test first using the following steps:

1. `cd` into the `npmregistry/` folder of this repo
2. run `npm config ls` an ensure that the `registry` value is set to `registry.npmjs.org`
3. run `npm install`. this should install successfully
4. Ensure that the `package-lock.json` includes `node_modules/jsdom` and that it includes the `peerDependenciesMeta` object. 

Now run the Code Artifact tests using the following steps:

1. ensure that your have a CodeArtifact registry that mirrors `registry.npmjs.org`
2. `cd` into the `codeartifact/` folder of this repo
3. run the command to login to the CodeArtfact repository
4. run `npm install` It is expected that this will fail. 

Switching the `registry` back to `registry.npmjs.org` will permit `jsdom` to install, but you'll now find that the `peerDependenciesMeta` object is now missing in the `package-lock.json` file. 


## Expanded package-lock.json details

### Via registry.npmjs.org 

```json

"node_modules/jsdom": {
      "version": "16.7.0",
      "resolved": "https://registry.npmjs.org/jsdom/-/jsdom-16.7.0.tgz",
      "integrity": "sha512-u9Smc2G1USStM+s/x1ru5Sxrl6mPYCbByG1U/hUmqaVsm4tbNyS7CicOSRyuGQYZhTu0h84qkZZQ/I+dzizSVw==",
      "dependencies": {
        "abab": "^2.0.5",
        "acorn": "^8.2.4",
        "acorn-globals": "^6.0.0",
        "cssom": "^0.4.4",
        "cssstyle": "^2.3.0",
        "data-urls": "^2.0.0",
        "decimal.js": "^10.2.1",
        "domexception": "^2.0.1",
        "escodegen": "^2.0.0",
        "form-data": "^3.0.0",
        "html-encoding-sniffer": "^2.0.1",
        "http-proxy-agent": "^4.0.1",
        "https-proxy-agent": "^5.0.0",
        "is-potential-custom-element-name": "^1.0.1",
        "nwsapi": "^2.2.0",
        "parse5": "6.0.1",
        "saxes": "^5.0.1",
        "symbol-tree": "^3.2.4",
        "tough-cookie": "^4.0.0",
        "w3c-hr-time": "^1.0.2",
        "w3c-xmlserializer": "^2.0.0",
        "webidl-conversions": "^6.1.0",
        "whatwg-encoding": "^1.0.5",
        "whatwg-mimetype": "^2.3.0",
        "whatwg-url": "^8.5.0",
        "ws": "^7.4.6",
        "xml-name-validator": "^3.0.0"
      },
      "engines": {
        "node": ">=10"
      },
      "peerDependencies": {
        "canvas": "^2.5.0"
      },
      "peerDependenciesMeta": {
        "canvas": {
          "optional": true
        }
      }
    },

```

### From Code Artifact 

```json

"node_modules/jsdom": {
      "version": "16.7.0",
      "resolved": "https://registry.npmjs.org/jsdom/-/jsdom-16.7.0.tgz",
      "integrity": "sha512-u9Smc2G1USStM+s/x1ru5Sxrl6mPYCbByG1U/hUmqaVsm4tbNyS7CicOSRyuGQYZhTu0h84qkZZQ/I+dzizSVw==",
      "dependencies": {
        "abab": "^2.0.5",
        "acorn": "^8.2.4",
        "acorn-globals": "^6.0.0",
        "cssom": "^0.4.4",
        "cssstyle": "^2.3.0",
        "data-urls": "^2.0.0",
        "decimal.js": "^10.2.1",
        "domexception": "^2.0.1",
        "escodegen": "^2.0.0",
        "form-data": "^3.0.0",
        "html-encoding-sniffer": "^2.0.1",
        "http-proxy-agent": "^4.0.1",
        "https-proxy-agent": "^5.0.0",
        "is-potential-custom-element-name": "^1.0.1",
        "nwsapi": "^2.2.0",
        "parse5": "6.0.1",
        "saxes": "^5.0.1",
        "symbol-tree": "^3.2.4",
        "tough-cookie": "^4.0.0",
        "w3c-hr-time": "^1.0.2",
        "w3c-xmlserializer": "^2.0.0",
        "webidl-conversions": "^6.1.0",
        "whatwg-encoding": "^1.0.5",
        "whatwg-mimetype": "^2.3.0",
        "whatwg-url": "^8.5.0",
        "ws": "^7.4.6",
        "xml-name-validator": "^3.0.0"
      },
      "engines": {
        "node": ">=10"
      },
      "peerDependencies": {
        "canvas": "^2.5.0"
      }
    },

```