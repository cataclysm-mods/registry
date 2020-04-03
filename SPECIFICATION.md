# The C:DDA Mod Specification

## Version `0.1`

This is a Request For Comments on the [Cataclysm: Dark Days Ahead](https://github.com/CleverRaven/Cataclysm-DDA) (abbreviated as `C:DDA`) mod specification for `modinfo.josn` files.
Please open discussions in the issues list, and send pull requests to patch this document and associated files.

This document, and all associated files, are licensed under the CC0 1.0 Universal license.

The key words "*must*", "*must not*", "*required*", "*shall*", "*shall not*", "*should*", "*should not*", "*recommended*", "*may*" and "*optional*" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Introduction

C:DDA is an [open source](https://en.wikipedia.org/wiki/Open-source_model) [rougelike](https://en.wikipedia.org/wiki/Roguelike) game with a [modding system](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/MODDING.md) that aspires to be purely data driven by [JSON](https://en.wikipedia.org/wiki/JSON) files provided by the game community. C:DDA itself provides a [loose specification](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INFO.md) on how these files are structured and how the C:DDA game engine interprets them.

However, in an effort to reduce development overhead and complexity of the core game code there are a number of things C:DDA does not provide by design:

1. C:DDA has no networking code. \
   This requires that any updating of the core game or associated community content must be managed outside C:DDA by either the user or a program run by the user.
2. C:DDA does not provide a formal schema of JSON files it interprets. \
   Due to the way the game's JSON parsing logic is implemented, the definition of what JSON C:DDA will correctly interpret is heavily dependent on non-trivial, performance sensitive C++ code. This makes it difficult for core game development to derive parsing logic from a pre-defined specification.
3. C:DDA does not provide it's own development tooling for the core game code or associated mods. \
   While the C:DDA repository leverages a number of general purpose [development tools](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/DEVELOPER_TOOLING.md), it does not provide project specific tooling to aide in development or debugging of the game itself.

**The primary goal of this document is increase the volume and technical quality content contributions to C:DDA to by providing a formal specifciation of JSON data interpreted by C:DDA.**

Once a formal specification is in place, it can be used to:

* Assist C:DDA contributors in understanding how C:DDA interprets JSON data
* Enable tool authors to build richer tooling for development of C:DDA content
* Enable the community to build more sophisticated tools to manage and extend C:DDA installations

## Design

The fundamental design of the C:DDA mod specification is as follows:

1. Each *distribution* (a mod and its associated files):
   * *must* be comprised of a single directory
   * *must* have an associated meta-data file that describes its contents named `modinfo.json` at the root of the *distribution* directory (e.g. `my-mod/metadata.json`)
   * *may* contain any number of sub-directories with additional JSON data files
2. The meta-data file *must* be detachable from the distribution itself. \
   This facilities easy building of indexes, and means meta-data can be created independently of the distribution itself, easing adoption by authors.
3. The meta-data file *must* be included in the distribution to facilitate easier indexing. \
   C:DDA uses the meta-data file to display information about a mod to players interacting with the C:DDA UI.

Presently the C:DDA mod specification repository is
[hosted on github](https://github.com/cataclysm-mods/registry).

## Validation

A [JSON Schema](schemas/modinfo.json) is provided for validation purposes. Any C:DDA distribution meta-data file *must* conform to this schema to be considered valid.

## The `modinfo.json` file

A `modinfo.json` file is designed to contain all the relevant meta-info about a mod, including its name, license, download location, dependencies, compatible versions of C:DDA, and the like. All C:DDA data files are simply JSON files using UTF-8 as character-encoding.

Except where stated otherwise all strings *should* be printable unicode only.

This metadata specification is heavy inspired by the
[The Comprehensive Kerbal Archive Network (CKAN) spec](https://github.com/KSP-CKAN/CKAN/blob/master/Spec.md), which in turn draws heavily on the 
[CPAN metadata spec](https://metacpan.org/pod/CPAN::Meta::Spec),
the [Debian Policy Manual](https://www.debian.org/doc/debian-policy/)
and the
[KSP-RealSolarSystem-Bundler](https://github.com/NathanKell/KSP-RealSolarSystem-Bundler).

### Example `modinfo.json` file

```json
{
    "spec_version"   : "0.1",
    "ident"          : "jury_rigged_robots",
    "name"           : "Jury-Rigged Robots",
    "description"    : "Options for salvaging, jury rigging, and reprogramming broken robots.",
    "download"       : "https://github.com/TheSalamanderDJ/Jury-Rigged-Robots/releases/download/1.1/Jury_Rigged_Robots.zip",
    "license"        : "gpl-3.0",
    "version"        : "1.1",
    "release_status" : "stable",
    "cdda_version"   : "0.D",
    "resources" : {
        "homepage"     : "https://discourse.cataclysmdda.org/t/jury-rigged-robots-mod/9943",
        "repository"   : "https://github.com/TheSalamanderDJ/Jury-Rigged-Robots"
    },
    "dependencies" : [ "dda" ]
}
```

#### Mandatory fields

##### spec_version

The version string of the C:DDA mod specification used to create this `modinfo.json` file. This string is the minimum version of the C:DDA mod specification than can validate this file.

This document describes the CKAN specification '0.1'.

##### ident

This is the globally unique identifier for the mod, and is how the mod will be referred to by other mods and mod meta-data files. It may only consist of lowercase ASCII-letters, ASCII-digits and `-` (dash). Eg: `aeromancy` or `jury-rigged-robots`. This is the identifier that will be used whenever the mod is referenced (by `modinfo.json`'s `dependencies` section or elsewhere).

Identifiers must be both:

1. Unique
2. All lowercase for human consumption and case-ignorant systems. Example: `Jury-Rigged Robots` must always be expressed as `jury-rigged-robots`, but another module cannot assume the `jury-rigged-robots` identifier.

The only time different mods should inherit the same identifiers are when a mod is intended to be a replacement for an older version of the same mod.

Example: `Artyoms' Gun Emporium` was later succedded by `Artyoms' Gun Emporium - Reloaded`, both of which use the same mod identifier: `arts-guns`.

##### name

This is the human readable name of the mod, and may contain any printable characters. E.g. `Aëromancy` or `Jury-Rigged Robots`.

##### description

A short, one line description of the mod and what it does.

##### download

A `modinfo.json` file *must* include *one of* either `download` or `source`.

A fully formed URL, indicating where a machine may download an archive of the described version of the mod distribution.

Example: `https://github.com/TheSalamanderDJ/Jury-Rigged-Robots/releases/download/1.1/Jury_Rigged_Robots.zip`

##### source

A `modinfo.json` file *must* include *one of* either `download` or `source`.

A JSON object describing how to retreive a distribution from a Git source control repository.

The way we handle retreival of distributions from a source repository is heavily inspired by [Ruby's Bundler](https://bundler.io/guides/git.html).

Example `modinfo.json` with a `source` entry:

```json
{
    "spec_version"   : "0.1",
    "ident"          : "jury-rigged-robots",
    "name"           : "Jury-Rigged Robots",
    "description"    : "Options for salvaging, jury rigging, and reprogramming broken robots.",
    "source"         : {
        "url" : "https://github.com/TheSalamanderDJ/Jury-Rigged-Robots.git",
        "tag" :  "1.1"
    },
    "license"        : "gpl-3.0",
    "version"        : "1.1",
    "release_status" : "stable",
    "cdda_version"   : "0.D",
    "resources" : {
        "homepage"     : "https://discourse.cataclysmdda.org/t/jury-rigged-robots-mod/9943",
        "repository"   : "https://github.com/TheSalamanderDJ/Jury-Rigged-Robots"
    },
    "dependencies" : [ "dda" ]
}
```

##### url

A `https` URL to a Git repository containing the distribution.

Example: `https://github.com/TheSalamanderDJ/Jury-Rigged-Robots.git`

##### branch

A `source` entry *must* contain *one of* either `branch`, `tag`, or `ref`.

The name of the Git branch in the repository containing the desired version of the distribution.

##### tag

A `source` entry *must* contain *one of* either `branch`, `tag`, or `ref`.

The name of the Git tag in the repository containing the desired version of the distribution.

##### ref

A `source` entry *must* contain *one of* either `branch`, `tag`, or `ref`.

The a valid Git reference in the repository containing the desired version of the distribution. This is typically a Git SHA, but may be anything valid according to [gitrevisions](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/gitrevisions.html#_specifying_revisions)

##### license

The license or list of licenses under which the mod is released. The same rules as per the
[Debian license specification](https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/#license-specification) apply, with the following modifications:

* The `MIT` license is always taken to mean the [Expat license](https://www.debian.org/legal/licenses/mit). See [MIT License: Variants](https://en.wikipedia.org/wiki/MIT_License#Variants) for details on why this matters.
* The creative commons licenses are permitted without a version number, indicating the author did not specify which version applies.
* Stripping of trailing zeros is not recognised.

The following license strings are also valid and indicate other licensing not described above:

- `open-source`: Other Open Source Initiative (OSI) approved license
- `restricted`: Requires special permission from copyright holder
- `unrestricted`: Not an OSI approved license, but not restricted
- `unknown`: License not provided in metadata

A single license, or list of licenses may be provided. The following are both valid, the first describing a mod released under the BSD license, the second under the *user's choice* of BSD-2-clause or GPL-2.0 licenses.

```
"license" : "BSD-2-clause"
"license" : [ "BSD-2-clause", "GPL-2.0" ]
```

If different assets in the mod have different licenses, the *most restrictive* license should be specified, which may be `restricted`.

##### version

The version of the mod. Versions have the format `[epoch:]mod_version`.

###### epoch

`epoch` is a single (generally small) unsigned integer. It may be omitted, in which case zero is assumed.

It is provided to allow mistakes in the version numbers of older versions of a package, and also a package's previous version numbering schemes, to be left behind.

###### mod_version

The `mod_version` portion of  `mod_version` is the main part of the version number. It is usually the version number of the original mod from which the `modinfo.json` file is created. Usually this will be in the same format as that specified by the mod author(s); however, it may need to be reformatted to fit into the package management system's format and comparison scheme.

The comparison behavior of the package management system with respect to the `mod_version` is described below. The `mod_version` portion of the version number is mandatory.

Version names *must* be ASCII-letters, ASCII-digits, and the characters `.` `+` `-` `_` (full stop, plus, dash, underscore) and should start with a digit.

`mod_version` examples:
```
"mod_version": "0.1_alpha"
"mod_version": "1.0"
"mod_version": "0:1.0.1+bugfix2"
"mod_version": "1:0.2-version-rework"
"mod_version": "1:1.0"
```

###### version ordering

When comparing two version numbers, first the `epoch` of each are compared, then the `mod_version` if `epoch` is equal. `epoch` is compared numerically. The `mod_version` part is compared by the package management system using the following algorithm:

The strings are compared from left to right.

First the initial part of each string consisting entirely of non-digit characters is determined. These two parts (one of which may be empty) are compared lexically. If a difference is found it is returned. The lexical comparison is a comparison of ASCII values modified so that all the letters sort earlier than all the non-letters.

Then the initial part of the remainder of each string which consists entirely of digit characters is determined. The numerical values of these two parts are compared, and any difference found is returned as the result of the comparison. For these purposes an empty string (which can only occur at the end of one or both version strings being compared) counts as zero.

These two steps (comparing and removing initial non-digit strings and initial digit strings) are repeated until a difference is found or both strings are exhausted.

Note that the purpose of epochs is to allow us to leave behind mistakes in version numbering, and to cope with situations where the version numbering scheme changes. It is not intended to cope with version numbers containing strings of letters which the package management system cannot interpret (such as ALPHA or pre-), or with silly orderings.

#### Optional fields

##### comment

A comment field, if included, is ignored. It is not displayed to users, nor used by programs. It's primary use is to convey information to humans examining the `modinfo.json` file manually.

##### authors

The author, or list of authors, for this mod. No restrictions are placed upon this field.

`author` examples:
```
"authors": "Damien <damien@mindglob.com>"
"authors": [ "ZhilkinSerg", "KorGgenT", "Damien <damien@mindglob.com>" ]
```

##### maintainers

Similar to `authors`.

The maintainer, or list of maintainers, for this mod. No restrictions are placed upon this field.

`maintainers` examples:
```
"maintainers": "remroy"
"maintainers": [ "kevingranade", "Coolthulhu" ]
```

##### description

A free form, long text description of the mod, suitable for displaying detailed information about the mod.

##### release_status

The release status of the mod, one of `stable`, `testing` or `development`, in order of decreasing stability.  If not specified, a value of `stable` is assumed.

##### cdda_version

The version of C:DDA this mod is targeting. This may be the string `"any"`, a C:DDA version string (eg: `"0.D"`). In the latter case, any release starting with the `cdda_version` is considered acceptable.

See [C:DDA Releases](https://cataclysmdda.org/releases/) for a list of stable version strings.

If no KSP target version is included, a default of `"any"` is assumed.

`cdda_version` examples:
```
# Target C:DDA 0.D "Danny" and all pre-releases prior to 0.E
"cdda_version": "0.D"

# Target C:DDA 0.B "Bring" and all pre-releases prior to 0.C
"cdda_version": "0.B"
```

##### cdda_version_min, cdda_version_max

The minimum and maximum version of C:DDA the mod requires to operate correctly. Same format as `cdda_version`. It is an error to include ether of these and the `cdda_version` field.

If version ranges specified by `cdda_version_min` and `cdda_version_max` are intended to include pre-releases between the min version and exclude all pre-releases after the max version.

`cdda_version_min` and `cdda_version_max` examples:
```
# Declare compatability with 0.C, 0.C experimental releases,
# and 0.D stable
"cdda_version_min": "0.C",
"cdda_version_min": "0.D"

# Declare compatability with 0.C, 0.D,
# 0.D experimental releases, and 0.E stable
"cdda_version_min": "0.C",
"cdda_version_min": "0.E"
```

#### resources

The `resources` field describes additional information that a user or program may wish to know about the mod, but which are not required for its installation or indexing. Presently the following fields are described. Unless specified otherwise, these are URLs:

- `homepage` : The preferred landing page for the mod.
- `bugtracker` : The mod's bugtracker if it exists.
- `license` : A link to the mod's liscense
- `repository` : The repository where the module source can be found.
- `ci` :  (**v1.6**) Continuous Integration (e.g. Jenkins) Server where the module is being built. `x_ci` is an alias used in netkan.
- `manual` : The mod's manual, if it exists.

Example resources:
```
"resources" : {
    "homepage"     : "https://github.com/CleverRaven/Cataclysm-DDA",
    "bugtracker"   : "https://github.com/CleverRaven/Cataclysm-DDA/issues",
    "repository"   : "https://github.com/CleverRaven/Cataclysm-DDA",
    "ci"           : "http://gorgon.narc.ro:8080/job/Cataclysm-Multijob/"
}
```

While all currently defined resources are URLs, future revisions of the spec may provide for more complex types.

It is permissible to have fields prefixed with an `x_`. These are considered custom use fields, and will be ignored. For example:

```
"x_twitter" : "https://twitter.com/pjf"
```

#### Special use fields

These fields are optional, and should only be used with good reason.
Typical mods *should not* include these special use fields.

##### download_size

If supplied, `download_size` is the number of bytes to expect when downloading from the `download` URL. It is recommended that this field is only generated by automated tools (where it is encouraged), and not filled in by hand.

##### download_hash

If supplied, `download_hash` is an object of hash digests. Currently SHA1 and SHA256 calculated hashes of the resulting file downloaded. It is recommended that this field is only generated by automated tools (where it is encouraged), and not filled in by hand.

```
"download_hash": {
    "sha1": "1F4B3F21A77D4A302E3417A7C7A24A0B63740FC5",
    "sha256": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855"
}
```

##### download_content_type

If supplied, `download_content_type` is the content type of the file downloaded from the `download` URL. It is recommended that this field is only generated by automated tools (where it is encouraged), and not filled in by hand.

```
"download_content_type": "application/zip"
```

#### Extensions

Any field starting with `x_` (an x, followed by an underscore) is considered
an *extension field*. The C:DDA mod tool-chain will *ignore* any such fields. These fields may be used to include additional machine or human-readable data in the files.

For example, one may have an `x_generated_by` field, to indicate it's the result of a custom build process.

Extension fields are unrestricted, and may contain any sort of data, including lists and objects.
