![Electron Installer for Red Hat](resources/logo.png)

# electron-installer-redhat [![Version](https://img.shields.io/npm/v/@dennisrijsdijk/electron-installer-redhat.svg)](https://www.npmjs.com/package/@dennisrijsdijk/electron-installer-redhat)

> Create a Red Hat package for your Electron app.


## Requirements

This tool requires Node 10 or greater and `rpmbuild` 4.13 or greater to build the `.rpm` package.

**Note**: RPM 4.13.0 or greater is required due to the [boolean dependency feature](http://rpm.org/user_doc/boolean_dependencies.html).

On Fedora you can do something like this:

```
$ sudo dnf install rpm-build
```

While on Debian/Ubuntu you'll need to do this instead:

```
$ sudo apt-get install rpm
```

## Installation

For use from command-line:

```
$ npm install -g @dennisrijsdijk/electron-installer-redhat
```

For use in npm scripts or programmatically:

```
$ npm install --save-dev @dennisrijsdijk/electron-installer-redhat
```


## Usage

Say your Electron app lives in `path/to/app`, and has a structure like this:

```
.
├── LICENSE
├── README.md
├── node_modules
│   ├── electron-packager
│   └── electron
├── package.json
├── resources
│   ├── Icon.png
│   ├── IconTemplate.png
│   └── IconTemplate@2x.png
└── src
    ├── index.js
    ├── main
    │   └── index.js
    └── renderer
        ├── index.html
        └── index.js
```

You now run `electron-packager` to build the app for Red Hat:

```
$ electron-packager . app --platform linux --arch x64 --out dist/
```

And you end up with something like this in your `dist` folder:

```
.
└── dist
    └── app-linux-x64
        ├── LICENSE
        ├── LICENSES.chromium.html
        ├── content_shell.pak
        ├── app
        ├── icudtl.dat
        ├── libgcrypt.so.11
        ├── libnode.so
        ├── locales
        ├── natives_blob.bin
        ├── resources
        ├── snapshot_blob.bin
        └── version
```

How do you turn that into a Red Hat package that your users can install?

### Command-Line

If you want to run `@dennisrijsdijk/electron-installer-redhat` straight from the command-line, install the package globally:

```
$ npm install -g @dennisrijsdijk/electron-installer-redhat
```

And point it to your built app:

```
$ npx --no-install @dennisrijsdijk/electron-installer-redhat --src dist/app-linux-x64/ --dest dist/installers/ --arch x86_64
```

You'll end up with the package at `dist/installers/app-0.0.1-1.x86_64.rpm`.

### Scripts

If you want to run `electron-installer-redhat` through npm, install the package locally:

```
$ npm install --save-optional @dennisrijsdijk/electron-installer-redhat
```

Edit the `scripts` section of your `package.json`:

```json
{
  "name": "app",
  "description": "An awesome app!",
  "version": "0.0.1",
  "scripts": {
    "start": "electron .",
    "build": "electron-packager . app --platform linux --arch x64 --out dist/",
    "rpm64": "@dennisrijsdijk/electron-installer-redhat --src dist/app-linux-x64/ --dest dist/installers/ --arch x86_64"
  },
  "devDependencies": {
    "electron-packager": "*",
    "electron-prebuilt": "*"
  },
  "optionalDependencies": {
    "@dennisrijsdijk/electron-installer-redhat": "^3.4.2"
  }
}
```

And run the script:

```
$ npm run rpm64
```

You'll end up with the package at `dist/installers/app-0.0.1-1.x86_64.rpm`.

### Programmatically

Install the package locally:

```shell
$ npm install --save-optional @dennisrijsdijk/electron-installer-redhat
```

And write something like this:

```javascript
const installer = require('@dennisrijsdijk/electron-installer-redhat')

const options = {
  src: 'dist/app-linux-x64/',
  dest: 'dist/installers/',
  arch: 'x86_64'
}

async function main (options) {
  console.log('Creating package (this may take a while)')

  try {
    await installer(options)
    console.log(`Successfully created package at ${options.dest}`)
  } catch (err) {
    console.error(err, err.stack)
    process.exit(1)
  }
}
main(options)
```

You'll end up with the package at `dist/installers/app-0.0.1-1.x86_64.rpm`.

_Note: As of 2.0.0, the Node-style callback pattern is no longer available. You can use [`util.callbackify`](https://nodejs.org/api/util.html#util_util_callbackify_original) if this is required for your use case._

### Options

Even though you can pass most of these options through the command-line interface, it may be easier to create a configuration file:

```javascript
{
  "dest": "dist/installers/",
  "icon": "resources/Icon.png",
  "categories": [
    "Utility"
  ]
}
```

And pass that instead with the `config` option:

```shell
$ npx --no-install @dennisrijsdijk/electron-installer-redhat --src dist/app-linux-x64/ --arch x86_64 --config config.json
```

Anyways, here's the full list of options:

#### src
Type: `String`
Default: `undefined`

Path to the folder that contains your built Electron application.

#### dest
Type: `String`
Default: `undefined`

Path to the folder that will contain your Red Hat installer.

#### rename
Type: `Function`
Default: `function (dest, src) { return path.join(dest, src); }`

Function that renames all files generated by the task just before putting them in your `dest` folder.

#### options.name
Type: `String`
Default: `package.name`

Name of the package (e.g. `atom`), used in the [`Name` field of the `spec` file](https://fedoraproject.org/wiki/Packaging:NamingGuidelines).

#### options.productName
Type: `String`
Default: `package.productName || package.name`

Name of the application (e.g. `Atom`), used in the [`Name` field of the `desktop` specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

#### options.genericName
Type: `String`
Default: `package.genericName || package.productName || package.name`

Generic name of the application (e.g. `Text Editor`), used in the [`GenericName` field of the `desktop` specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

#### options.description
Type: `String`
Default: `package.description`

Short, one-line description of the application; do not end with a period.
Used in the [`Summary` field of the `spec` file](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

#### options.productDescription
Type: `String`
Default: `package.productDescription || package.description`

Long description of the application, used in the [`%description` tag of the `spec` file](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

#### options.version
Type: `String`
Default: `package.version`

Version number of the package, used in the [`Version` field of the `spec` file](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

#### options.revision
Type: `String`
Default: `package.revision || 1`

Revision number of the package, used in the [`Release` field of the `spec` file](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

#### options.license
Type: `String`
Default: `package.license`

License of the package, used in the [`License` field of the `spec` file](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

#### options.arch
Type: `String`
Default: `undefined`

Machine architecture the package is targeted to, used to set the `--target` option.

#### options.platform
Type: `String`
Default: Operating system platform of the host machine. For possible values see Node.js [process.platform](https://nodejs.org/api/process.html#process_process_platform)

Operating system platform the package is targeted to, used to set the [`--target`](https://linux.die.net/man/8/rpmbuild) option.

#### options.requires
Type: `Array[String]`
Default: The minimum list of packages needed for Electron to run

Packages that are required when the program starts, used in the [`Requires` field of the `spec` file](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

All user requirements will be appended to the default array of requirements, and any duplicates will be removed.

#### options.homepage
Type: `String`
Default: `package.homepage || package.author.url`

URL of the homepage for the package, used in the [`URL` field of the `spec` specification](https://docs.fedoraproject.org/en-US/quick-docs/creating-rpm-packages/index.html#con_rpm-spec-file-overview).

#### options.compressionLevel
Type: `Number`
Default: `2`

Package compression level, from `0` to `9`.

#### options.bin
Type: `String`
Default: `package.name`

Relative path to the executable that will act as binary for the application, used in the [`Exec` field of the `desktop` specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

The generated package will contain a symlink `/usr/bin/<%= options.name %>` pointing to the path provided here.

For example, providing this configuration:

```js
{
  src: '...',
  dest: '...',
  name: 'foo',
  bin: 'resources/cli/launcher.sh'
}
```

Will create a package with the following symlink:

```
usr/bin/foo@ -> ../share/foo/resources/cli/launcher/sh
```

And a desktop specification with the following `Exec` key:

```
Exec=foo %U
```

#### options.execArguments
Type: `Array[String]`
Default: `[]`

Command-line arguments to pass to the executable. Will be added to the [`Exec` field of the `desktop` specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

#### options.icon
Type: `String` or `Object[String:String]`
Default: `undefined`

Path to a single image that will act as icon for the application:

```js
{
  icon: 'resources/Icon.png'
}
```

Or multiple images with their corresponding resolutions:

```js
{
  icon: {
    '48x48': 'resources/Icon48.png',
    '64x64': 'resources/Icon64.png',
    '128x128': 'resources/Icon128.png',
    '256x256': 'resources/Icon256.png',
    'scalable': 'resources/Icon.svg',
    'symbolic': 'resources/Icon-symbolic.svg',
  }
}
```
Per the [icon theme specification](https://specifications.freedesktop.org/icon-theme-spec/icon-theme-spec-0.11.html), image files must either PNGs or SVGs. The SVG format can only be used for the `scalable` or `symbolic` resolutions.


#### options.categories
Type: `Array[String]`
Default: `[]`

Categories in which the application should be shown in a menu, used in the [`Categories` field of the `desktop` specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

For possible values check out the [Desktop Menu Specification](http://standards.freedesktop.org/menu-spec/latest/apa.html).

#### options.mimeType
Type: `Array[String]`
Default: `[]`

MIME types the application is able to open, used in the [`MimeType` field of the `desktop` specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).

If this option is specified, make sure to run `update-desktop-database &> /dev/null` as part of the `post` and `postun` scripts to refresh the [cached database of MIME types](https://fedoraproject.org/wiki/NewMIMESystem).


#### options.scripts
Type: `Object[String:String]`
Default: `undefined`

Path to [installation scripts](https://docs.fedoraproject.org/en-US/packaging-guidelines/Scriptlets/) with their corresponding name. The files contents will be added to the spec file.

```javascript
{
  scripts: {
    'pre': 'resources/pre_script',
    'post': 'resources/post_script',
    'preun': 'resources/preun_script',
    'postun': 'resources/postun_script'
  }
}
```

#### options.desktopTemplate
Type: `String`
Default: [`resources/desktop.ejs`](https://github.com/dennisrijsdijk/electron-installer-redhat/blob/main/resources/desktop.ejs)

The absolute path to a custom template for the generated [FreeDesktop.org desktop entry](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html) file.

#### options.specTemplate
Type: `String`
Default: [`resources/spec.ejs`](https://github.com/dennisrijsdijk/electron-installer-redhat/blob/main/resources/spec.ejs)

The absolute path to a custom template for the generated [SPEC file](https://rpm-packaging-guide.github.io/#what-is-a-spec-file).

## Meta

* Code: `git clone git://github.com/dennisrijsdijk/electron-installer-redhat.git`
* Home: <https://github.com/dennisrijsdijk/electron-installer-redhat/>


## Contributors

* Daniel Perez Alvarez ([unindented@gmail.com](mailto:unindented@gmail.com))
* Dennis Rijsdijk ([hello@dennis.gg](mailto:hello@dennis.gg))


## License

Copyright (c) 2016 Daniel Perez Alvarez ([unindented.org](https://unindented.org/)). This is free software, and may be redistributed under the terms specified in the LICENSE file.
