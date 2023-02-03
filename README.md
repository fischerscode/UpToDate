[![CI](https://github.com/fischerscode/uptodate/actions/workflows/ci.yaml/badge.svg)](https://github.com/fischerscode/uptodate/actions/workflows/ci.yaml)
[![](https://img.shields.io/github/v/release/fischerscode/uptodate)](https://github.com/fischerscode/uptodate/releases/latest)
[![](https://img.shields.io/github/license/fischerscode/uptodate)](https://github.com/fischerscode/uptodate/blob/master/LICENSE)
[![on release](https://github.com/fischerscode/uptodate/actions/workflows/release.yaml/badge.svg)](https://github.com/fischerscode/uptodate/actions/workflows/release.yaml)

You can use UpToDate either as a [command line tool](#uptodate-as-a-command-line-tool) or a [GitHub Action](#uptodate-action)


# Content
- [Content](#content)
- [UpToDate Action](#uptodate-action)
  - [Usage](#usage)
- [UpToDate as a command line tool](#uptodate-as-a-command-line-tool)
  - [Usage](#usage-1)
- [Configuration](#configuration)
  - [Config file](#config-file)
  - [Dependency types](#dependency-types)
    - [Web dependency](#web-dependency)
    - [Json web dependency](#json-web-dependency)
    - [Yaml web dependency](#yaml-web-dependency)
    - [GitHub dependency](#github-dependency)
    - [Helm dependency](#helm-dependency)

# UpToDate Action
This action helps you to keep your repository up to date, by creating issues as soon as your dependencies changed.

## Usage
1. Create a workflow like this:
```yaml
name: uptodate

on:
  schedule:
    - cron: "0 0 * * *"

jobs:
  uptodate:
    runs-on: ubuntu-latest
    steps:
      # You have to check out your repo first.
      - uses: actions/checkout@v2
      - uses: fischerscode/uptodate@v0.7
        with:
          # The location of the config file.
          # Defaults to '.uptodate.yaml'
          config: ''

          # The token used for creating issues.
          # Defaults to ${{ github.token }}
          token: ''

          # The target repository for new issues.
          # Defaults to ${{ github.repository }}
          repository: ''
```
2. Create the config file `.uptodate.yaml` ([Documentation](#configuration)).

# UpToDate as a command line tool

`dart pub global activate uptodate`

OR

Use a  [prebuilt executable](https://github.com/fischerscode/uptodate/releases/latest).

## Usage
```
A tool that helps you to keep your repository up to date.

Usage: uptodate <command> [arguments]

Global options:
-h, --help                Print this usage information.
-f, --file (mandatory)    The config file.
-v, --[no-]verbose        Detailed output.

Available commands:
  check     Check the dependencies in the config file for updates.
  github    Check for updates and create issues in your GitHub repository.
  version   Print uptodate version.

Run "uptodate help <command>" for more information about a command.
```

# Configuration
Uptodate is configured using a [single file](#config-file) where all the [dependencies]() and their currently used versions are stored in.

## Config file
```yaml
dependencies:   # The unsorted list containing the dependencies.
  - name: testdependency        # The name of the dependency
    type: typ                   # The type of the dependency
    currentVersion: 1.2.3       # The current version of the dependency
    issueLabels:                # The labels added to the issue
      - dependencies
      - test
defaultIssueTitle: "Update $name to $latestVersion"     # The default issue title
defaultIssueBody: "Update $name to $latestVersion"      # The default issue body
```
In `defaultIssueTitle` and `defaultIssueBody` the following variables are allowed:
- `$name`: the name of the dependency
- `$currentVersion`: the current version of the dependency
- `$latestVersion`: the latest available version

Using `issueTitle` every dependency can be override `defaultIssueTitle`.
Using `issueBody` every dependency can be override `defaultIssueBody`.

## Dependency types
There are multiple dependency types:
 - [Web dependency](#web-dependency)
 - [Json web dependency](#json-web-dependency)
 - [Yaml web dependency](#yaml-web-dependency)
 - [GitHub dependency](#github-dependency)
 - [Helm dependency](#helm-dependency)

### Web dependency
An url gets called to receive the latest version of the dependency.
```yaml
dependencies:
  - name: testdependency        # The name of the dependency. (required)
    type: web                   # The type of the dependency. (required)
    currentVersion: 1.2.3       # The current version of the dependency. (required)
    url: 'http://example.com'   # The url that gets called. (required)
```
### Json web dependency
An url gets called to receive a json document containing the latest version.
```yaml
dependencies:
  - name: testdependency          # The name of the dependency. (required)
    type: webjson                 # The type of the dependency. (required)
    currentVersion: 1.2.3         # The current version of the dependency. (required)
    url: 'http://example.com'     # The url that gets called. (required)
    path: args.versions.0.version # The path to the latest version. (defaults to '')
    prefix: v                     # The prefix of the semantic versions. 
                                  # (v1.2.3 instead of 1.2.3)
                                  # Defaults to ''
```
### Yaml web dependency
An url gets called to receive a (single) yaml document containing the latest version.
```yaml
dependencies:
  - name: testdependency          # The name of the dependency. (required)
    type: webyaml                 # The type of the dependency. (required)
    currentVersion: 1.2.3         # The current version of the dependency. (required)
    url: 'http://example.com'     # The url that gets called. (required)
    path: args.versions.0.version # The path to the latest version. (defaults to '')
    prefix: v                     # The prefix of the semantic versions. 
                                  # (v1.2.3 instead of 1.2.3)
                                  # Defaults to ''
```
### GitHub dependency
Uses a GitHub repository to receive the latest version.
The version is gathered from the latest (non pre-)release (or tag) that has the right prefix.
```yaml
dependencies:
  - name: gitdependency            # The name of the dependency. (required)
    type: github                   # The type of the dependency. (required)
    currentVersion: v1.2.3         # The current version of the dependency. (required)
    repo: 'fischerscode/uptodate'  # The repository. (required)
    isTag: true                    # Wether the version is determined by the latest
                                   # tag rather then release (defaults to false)
    path: tag_name                 # The path to the latest version. (defaults to tag_name)
    prefix: v                      # The prefix of the semantic versions. 
                                   # Is set, releases/tags not matching the prefix are ignored.
                                   # (v1.2.3 instead of 1.2.3)
                                   # Defaults to ''
```
### Helm dependency
Uses a Helm repository to receive the latest version.
The version is gathered from by the yaml path.
```yaml
dependencies:
  - name: helmdependency           # The name of the dependency. (required)
    type: helm                     # The type of the dependency. (required)
    currentVersion: v1.2.3         # The current version of the dependency. (required)
    repo: 'https://helm.traefik.io/traefik'  # The repository. (required)
    chart: traefik                      # The chart. (required)
    prefix: v                           # The prefix of the semantic versions. 
                                        # (v1.2.3 instead of 1.2.3)
                                        # Defaults to ''
    path: 'entries.traefik.0.version'   # The path to the latest version. (defaults to 'entries.$chart.0.version')
```