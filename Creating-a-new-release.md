This repository uses [AppVeyor](https://ci.appveyor.com/project/springcomp/optimized-azerty-win) continuous integration to build and package artifacts that must be published on each new release. The release and its assets are created automatically with the correct version.

The release must now be modified to include:

- The fixed `AZERTY NF Z71-300 Keyboard Layout` title.
- A change log.
- A link to the french documentation. 

The documentation pages must also be updated to refer to the new `setup.zip` download.

## Updating the Release

In order to create a new release, please follow the steps outlined here:

1. Push a version tag to the current HEAD of the `master` branch.  
   Use a version tag with the following format: v _M_._m_._x_._y_.  
   `git tag v1.2.0.0 master` __important__: make sure to specify the base ref branch `master`.

2. This will trigger a new build and create a new release on the [Releases](https://github.com/springcomp/optimized-azerty-win/releases) page.

3. Move the french documentation notice from the previous release to the current release.

## Updating the Documentation

4. Copy the link to the `setup.zip` release asset in the Windows clipboard.
5. Use [Bitly](https://bitly.com/) or another service to shorten the copied URL.
6. Modify the `download.md` file in the `gh-pages` documentation branch, to update the download link from the PowerShell sample download instructions.
7. Push those updated changes to the `gh-pages` branch.