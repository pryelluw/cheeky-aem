

# Sample AEM project template

This is a project template for AEM-based applications. It is intended as a best-practice set of examples as well as a potential starting point to develop your own functionality.

## Archetype

```
mvn -B archetype:generate \
 -D aemVersion=6.5.0 \
 -D archetypeGroupId=com.adobe.aem \
 -D archetypeArtifactId=aem-project-archetype \
 -D archetypeVersion=25 \
 -D appTitle="Cheeky" \
 -D appId="cheeky" \
 -D groupId="com.cheeky"
```

## Issues installing

Got npm error:

```
388 verbose stack Error: tsconfig-paths-webpack-plugin@3.4.0 postinstall: `husky install`
388 verbose stack spawn ENOENT
388 verbose stack     at ChildProcess.<anonymous> (/Users/privera/.nvm/versions/node/v10.13.0/lib/node_modules/npm/node_modules/npm-lifecycle/lib/spawn.js:48:18)
388 verbose stack     at ChildProcess.emit (events.js:182:13)
388 verbose stack     at maybeClose (internal/child_process.js:962:16)
388 verbose stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:251:5)
389 verbose pkgid tsconfig-paths-webpack-plugin@3.4.0
390 verbose cwd /Users/privera/Documents/celerity/aem-learning/cheeky-aem/cheeky/ui.frontend
391 verbose Darwin 19.4.0
392 verbose argv "/Users/privera/.nvm/versions/node/v10.13.0/bin/node" "/Users/privera/.nvm/versions/node/v10.13.0/bin/npm" "install"
393 verbose node v10.13.0
394 verbose npm  v6.4.1
395 error file sh
396 error code ELIFECYCLE
397 error errno ENOENT
398 error syscall spawn
399 error tsconfig-paths-webpack-plugin@3.4.0 postinstall: `husky install`
399 error spawn ENOENT
400 error Failed at the tsconfig-paths-webpack-plugin@3.4.0 postinstall script.
400 error This is probably not a problem with npm. There is likely additional logging output above.
401 verbose exit [ 1, true ]
```


I was able to install it by doing the following:

You need `node v 10.13.0`

`cd` into `ui.frontend`

run `yarn install`

This will fail with error:

```
error aem-clientlib-generator@1.7.5: The engine "node" is incompatible with this module. Expected version ">=10.19.0". Got "10.13.0"
error Found incompatible module.
```

run `npm install`

This should succeed.

Afterwards, I ran `mvn clean install` and it worked as expected.

## Modules

The main parts of the template are:

* core: Java bundle containing all core functionality like OSGi services, listeners or schedulers, as well as component-related Java code such as servlets or request filters.
* it.tests: Java based integration tests
* ui.apps: contains the /apps (and /etc) parts of the project, ie JS&CSS clientlibs, components, and templates
* ui.content: contains sample content using the components from the ui.apps
* ui.config: contains runmode specific OSGi configs for the project
* ui.frontend: an optional dedicated front-end build mechanism (Angular, React or general Webpack project)
* ui.tests: Selenium based UI tests
* all: a single content package that embeds all of the compiled modules (bundles and content packages) including any vendor dependencies
* analyse: this module runs analysis on the project which provides additional validation for deploying into AEMaaCS

## How to build

To build all the modules run in the project root directory the following command with Maven 3:

    mvn clean install

To build all the modules and deploy the `all` package to a local instance of AEM, run in the project root directory the following command:

    mvn clean install -PautoInstallSinglePackage

Or to deploy it to a publish instance, run

    mvn clean install -PautoInstallSinglePackagePublish

Or alternatively

    mvn clean install -PautoInstallSinglePackage -Daem.port=4503

Or to deploy only the bundle to the author, run

    mvn clean install -PautoInstallBundle

Or to deploy only a single content package, run in the sub-module directory (i.e `ui.apps`)

    mvn clean install -PautoInstallPackage

## Testing

There are three levels of testing contained in the project:

### Unit tests

This show-cases classic unit testing of the code contained in the bundle. To
test, execute:

    mvn clean test

### Integration tests

This allows running integration tests that exercise the capabilities of AEM via
HTTP calls to its API. To run the integration tests, run:

    mvn clean verify -Plocal

Test classes must be saved in the `src/main/java` directory (or any of its
subdirectories), and must be contained in files matching the pattern `*IT.java`.

The configuration provides sensible defaults for a typical local installation of
AEM. If you want to point the integration tests to different AEM author and
publish instances, you can use the following system properties via Maven's `-D`
flag.

| Property | Description | Default value |
| --- | --- | --- |
| `it.author.url` | URL of the author instance | `http://localhost:4502` |
| `it.author.user` | Admin user for the author instance | `admin` |
| `it.author.password` | Password of the admin user for the author instance | `admin` |
| `it.publish.url` | URL of the publish instance | `http://localhost:4503` |
| `it.publish.user` | Admin user for the publish instance | `admin` |
| `it.publish.password` | Password of the admin user for the publish instance | `admin` |

The integration tests in this archetype use the [AEM Testing
Clients](https://github.com/adobe/aem-testing-clients) and showcase some
recommended [best
practices](https://github.com/adobe/aem-testing-clients/wiki/Best-practices) to
be put in use when writing integration tests for AEM.

## Static Analysis

The `analyse` module performs static analysis on the project for deploying into AEMaaCS. It is automatically
run when executing

    mvn clean install

from the project root directory. Additional information about this analysis and how to further configure it
can be found here https://github.com/adobe/aemanalyser-maven-plugin

### UI tests

They will test the UI layer of your AEM application using Selenium technology. 

To run them locally:

    mvn clean verify -Pui-tests-local-execution

This default command requires:
* an AEM author instance available at http://localhost:4502 (with the whole project built and deployed on it, see `How to build` section above)
* Chrome browser installed at default location

Check README file in `ui.tests` module for more details.

## ClientLibs

The frontend module is made available using an [AEM ClientLib](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/clientlibs.html). When executing the NPM build script, the app is built and the [`aem-clientlib-generator`](https://github.com/wcm-io-frontend/aem-clientlib-generator) package takes the resulting build output and transforms it into such a ClientLib.

A ClientLib will consist of the following files and directories:

- `css/`: CSS files which can be requested in the HTML
- `css.txt` (tells AEM the order and names of files in `css/` so they can be merged)
- `js/`: JavaScript files which can be requested in the HTML
- `js.txt` (tells AEM the order and names of files in `js/` so they can be merged
- `resources/`: Source maps, non-entrypoint code chunks (resulting from code splitting), static assets (e.g. icons), etc.

## Maven settings

The project comes with the auto-public repository configured. To setup the repository in your Maven settings, refer to:

    http://helpx.adobe.com/experience-manager/kb/SetUpTheAdobeMavenRepository.html
