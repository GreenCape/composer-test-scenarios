# Composer Test Scenarios

Manage multiple "test scenarios", each with a different set of Composer dependencies.

## Use Case

You will likely find this project to be useful when your project's composer.json file contains a requirement with multiple versions, such as:
```
    "require": {
        "symfony/console": "^2.8|^3|^4",
        ...
    }
```
If you wish to be able to easily test all of the different significant possible dependency resolutions for your project, the usual course of action is to use [lowest, current, highest testing](https://blog.wyrihaximus.net/2015/06/test-lowest-current-and-highest-possible-on-travis/). The basic purpose of each of these tests are as follows:

- **Current** tests run against the specific dependencies in a committed composer.lock file, ensuring that the dependencies being used do not change during the implementation of a new feature.

- **Highest** tests first run `composer update` to ensure that all dependencies are brought up to date before the tests are run. If the current tests are passing, but the highest tests fail, it is an indication that some dependency might have accidentally introduced a change that is not backwards compatible with previous versions.

- **Lowest** tests first run `composer update --prefer-lowest` to install the absolute lowest version permitted by the project's version constraints. If the lowest tests fail, but other tests pass, it indicates that the current feature may have introduced calls to some dependency APIs not available in the versions specified in the project's composer.json file.

If your composer.lock holds Symfony 3 components, then the `highest` test can be used to test Symfony 4, and the `lowest` test can be used for Symfony 2. In practice, though, it is difficult to craft a composer.json file that can easily be altered to run all three scenarios. Multipe alterations are needed, and the test scripts become more complicated. Furthermore, if you wish to do current / highest testing on both Symfony 3.4 and Symfony 4, then there is nothing for it but to commit multiple composer.lock files.

That is where this project comes in.

- Test scenarios are [defined in your composer.json file](#define-scenarios). Scenarios are named, and it is easy to add more.
- Once your test scenarios are defined, they are automatically kept up to date whenever `composer update` is ran.
- Test scenarios can be [referenced by name in your travis.yml test matrix](#install-scenarios), making it easy to read and modify your test suite.
- Any scenario can easily be [installed locally to allow ad-hoc testing](#test-locally) of different dependencies without needing to temporarily modify the project's main composer.json file.

As an added bonus, the test scripts in the example composer.json file automaticlly run a linter on all PHP files in the porject, and enforce PSR-2 coding conventions on the project source files. The coding standards may be customized to suit.

## Usage

To add these scripts to your project, run:
```
composer require greg-1-anderson/composer-test-scenarios:^1
```

#### Initial setup

Use the [example composer.json file](example-composer.json) as a guide to set up your project's composer.json file. In particular, you should:

- Copy the `scripts` section.
- If using coveralls.io, be sure to use the version constraints from the example composer.json file for satooshi/php-coveralls until [satooshi/php-coveralls#203](https://github.com/php-coveralls/php-coveralls/issues/203#issuecomment-347710771) is fixed.
- Set the platform PHP setting in the config section if you want your default test scenario to be based on Symfony 3.

#### Define scenarios

Alter the `create-scenario` commands in the `post-update-cmd` script to create the test scenarios you need.

- Call `create-scenario name` to create a test scenario with the specified name.
- Use additional arguments to list the Composer requirements to use in this scenario, e.g. `symfony/console:^2.8`
- Other flags are available to alter the scenario's composer.json file as needed:
  - `--platform-php 7.0`: set the platform php version (recommended). Default: unset.
  - `--stability stable`: set the stability. Default: stable
  - `--create-lockfile`: create a composer.lock file to commit. This is the default.
  - `--no-lockfile`: create a modified composer.json file, but omit the composer.lock file. You may specify this option for any scenario that has only **highest** or **lowest** tests. A lock file is required to do **current** tests.

#### Install scenarios

Use the [example .travis.yml]() file as a starting point for your tests. Alter the test matrix as necessary to test highest, current and/or lowest dependencies as desired for each of the scenarios. Any scenario referenced in your `.travis.yml` file must be defined in your `composer.json` file. The Travis test matrix will define the php version to use in the test, the scenario to use, and whether to do a lowest, current or highest test.

- Define the `SCENARIO` environment variable to name the scenario to test. If this variable is not defined, then the composer.json / composer.lock at the root of the project will be used for the test run.
- Use the `HIGHEST_LOWEST` environment variable to specify whether a lowest, current or highest test should be done.
  - To do a **highest** test, set `HIGHEST_LOWEST=update`.
  - To do a **lowest** test, set `HIGHEST_LOWEST="update --prefer-lowest"`.
  - To do a **current** test, do leave `HIGHEST_LOWEST` undefined.

With this setup, all that you need to do to create your scenarios is to run `composer update`. Commit the entire contents of the generated `scenarios` directory. Thereafter, every subsequent time that you run `composer update`, your scenario lock files will also be brought up to date. Commit the changes to your scenarios directory whenever you commit your updated `composer.lock` file.

#### Test locally

To do ad-hoc testing, run:
```
$ composer scenario symfony4
$ composer test
```
To go back to the default scenario, just run `composer install` (or `composer scenario`, which does the same thing).
