# Gitlab CI with Drupal 8

[![pipeline status master](https://gitlab.com/mog33/gitlab-ci-drupal/badges/master/pipeline.svg)](https://gitlab.com/mog33/gitlab-ci-drupal/commits/master)
[![pipeline status testing](https://gitlab.com/mog33/gitlab-ci-drupal/badges/master/pipeline.svg)](https://gitlab.com/mog33/gitlab-ci-drupal/commits/testing)

<img src="https://www.drupal.org/sites/all/themes/drupalorg_themes/blueprint/images/logo-d8.svg"  width="120" height="120"> +
<img src="https://about.gitlab.com/images/ci/gitlab-ci-cd-logo_2x.png"  width="120" height="120">

[Gitlab CI](https://docs.gitlab.com/ee/ci/README.html) for a
[Drupal 8](https://www.drupal.org) module / theme or project.

Include **Build**,
**Unit testing**, **Code quality**, **Metrics** and **Deploy** samples.

- [Prerequisites](#prerequisites)
- [Quick how to](#quick-how-to)
  - [Use Gitlab CI with your Drupal 8 module](#use-gitlab-ci-with-your-drupal-8-module)
  - [Use Gitlab CI with your full Drupal project](#use-gitlab-ci-with-your-full-drupal-project)
- [Usage](#usage)
  - [Skip jobs](#skip-jobs)
  - [Triggering pipeline](#triggering-pipeline)
- [Workflow proposed](#workflow-proposed)
  - [Branch master](#branch-master)
  - [Branch testing](#branch-testing)
- [Advanced usage](#advanced-usage)
  - [Nightwatch.js for Drupal 8](#nightwatchjs-for-drupal-8)
  - [Behat tests for Drupal 8](#behat-tests-for-drupal-8)
  - [PHPunit tests for Drupal 8](#phpunit-tests-for-drupal-8)
  - [Codecov.io support in PHPUNIT Code coverage](#codecovio-support-in-phpunit-code-coverage)
  - [Rules for linting / Code standards / QA](#rules-for-linting--code-standards--qa)
  - [Accessibility with Pa11y](#accessibility-with-pa11y)
- [Jobs detail](#jobs-detail)
- [CI image including tools](#ci-image-including-tools)
- [Running the jobs locally with Docker](#running-the-jobs-locally-with-docker)
- [Future plan](#future-plan)
- [Credits](#credits)

## Prerequisites

- Gitlab CI with a [runner that support docker](https://docs.gitlab.com/runner/)
- Minimal understanding of [Gitlab CI](https://about.gitlab.com/features/gitlab-ci-cd/) and [Gitlab CI Yaml](https://docs.gitlab.com/ee/ci/yaml)

## Quick how to

### Use Gitlab CI with your Drupal 8 module

Push your module to a Gitlab with CI and runners enabled.

[Gitlab.com](https://gitlab.com) offer 2,000 CI pipeline minutes/month on free
accounts.

- Copy `.gitlab-ci.yml` file and `.gitlab-ci` folder in the root of your Drupal module or theme (same level as `my_module_or_theme.info.yml` file).
- Go to Gitlab **Settings > CI/ CD > Variables** and add those variables:

```bash
CI_IMAGE_TYPE           selenium
CI_TYPE                 module
NIGHTWATCH_TESTS        --tag my_module
WEB_ROOT                /var/www/html
PHP_CODE                /var/www/html/modules/custom
SKIP_TEST_SECURITY      1
SKIP_TEST_BEHAT         1
SKIP_TEST_PA11Y         1
```

![gitlab-variables](https://gitlab.com/mog33/gitlab-ci-drupal/uploads/9a12c6590d5001ecde3330ff8af3a9c0/gitlab-variables.jpg)

- Create a branch **testing** and push to Gitlab.

Check your project pipeline or
[Run a pipeline from Gitlab UI](https://docs.gitlab.com/ee/ci/pipelines.html#manually-executing-pipelines)

- Create a branch **master** and push, see the pipeline running!

As an example you can check my module:
[Content moderation edit notify](https://gitlab.com/mog33/content_moderation_edit_notify)

### Use Gitlab CI with your full Drupal project

Push your project to a Gitlab with CI and runners enabled.

[Gitlab.com](https://gitlab.com) offer 2,000 CI pipeline minutes/month on free
accounts.

Assuming your project include a `composer.json` file from the [Drupal project
template](https://github.com/drupal-composer/drupal-project).

**Note**: should work with other Drupal distributions or project but this is not tested yet.

- Copy `.gitlab-ci.yml` file and `.gitlab-ci` folder in the root of your Drupal
project (same level as `composer.json` file).
- Put your code in the `web/modules/custom` and `web/themes/custom` folders of your project.
- Create a branch **testing** and push to Gitlab.

Check your project pipeline or
[Run a pipeline from Gitlab UI](https://docs.gitlab.com/ee/ci/pipelines.html#manually-executing-pipelines)

- Create a branch **master** and push, see the pipeline running!

As an example you can check my project on a Drupal 8 template:
[Drupal 8 project template](https://gitlab.com/mog33/drupal-composer-advanced-template)

## Usage

**Note**: The `.gitlab-ci.yml` file is meant to be a starting point for working
jobs with [Drupal 8](https://www.drupal.org), feel free to cherry pick what you
need but be careful about dependencies between some jobs and templates logic.

If your commit message contains **[ci skip]** or **[skip ci]**, using any
capitalization, the commit will be created but the pipeline will be skipped.

First look in `.gitlab-ci/.gitlab-ci-variables.yml` and check the variables.
This is all the variables you can override from global settings on Gitlab CI or
when manually running a pipeline.

See section [Advanced usage](#advanced-usage) for more details on each relation
between variables and jobs.

### Skip jobs

You can set variables values on Gitlab CI UI under _Settings > CI / CD > Variables_

With variables you can disable some tests without editing any file, available
skip variables are:

```yaml
  # Skip all tests jobs (next list).
  SKIP_TESTS: 0
  # Skip single jobs in tests. for tests information see
  # https://www.drupal.org/docs/8/testing/types-of-tests-in-drupal-8.
  #
  # Phpunit unit,kernel
  SKIP_TEST_UNITKERNEL: 0
  # Phpunit code coverage with optional Codecov.io support
  SKIP_TEST_CODECOVERAGE: 0
  # Phpunit functional tests (browser tests)
  SKIP_TEST_FUNCTIONAL: 0
  # Phpunit functional javascript tests (browser with javascript tests)
  SKIP_TEST_FUNCTIONALJS: 0
  # Nightwatch tests (browser with javascript tests), since Drupal 8.6
  # Currently not fully ready for Drupal.
  SKIP_TEST_NIGHTWATCH: 0
  # Symfony security check on composer.lock
  SKIP_TEST_SECURITY: 0
  # Behat tests
  SKIP_TEST_BEHAT: 0
  # Accessibility test
  SKIP_TEST_PA11Y: 0
  # Skip all QA jobs (Code sniffer with Drupal standards)
  SKIP_QA: 0
  # Skip all lint jobs (Javascript with eslint, Css with stylelint, Sass with sass-lint)
  SKIP_LINT: 0
  # Skip single lint sass job (Only one not included in Drupal core)
  SKIP_SASS_LINT: 0
  # Skip all metrics jobs (phpmetrics, phploc, pdepend)
  SKIP_METRICS: 0
  # Skip all deploy jobs by default (samples of deploy)
  SKIP_DEPLOY: 1
```

### Triggering pipeline

If you want to choose when to run the tests, you can adapt rules in
`.gitlab-ci.yml`, see
[Gitlab documentation](https://docs.gitlab.com/ee/ci/yaml/#only-and-except-simplified)

Tests (and Build) are by default on a branch `testing` and on all `tags`

```yaml
.test_except_only: &test_except_only
  except:
    refs:
      - master
    variables:
      - $SKIP_TESTS == "1"
  only:
    refs:
      - testing
      - tags
```

QA and Lint is run by default on all `branches`, you can adapt on each jobs

```yaml
.qa_template:
  only:
    refs:
      - branches
  except:
    variables:
      - $SKIP_QA == "1"
#...
.lint_template:
  only:
    refs:
      - branches
  except:
    variables:
      - $SKIP_LINT == "1"
```

Metrics jobs are by default on each push on `master` and all `tags`

```yaml
.metrics_template:
  only:
    refs:
      - master
      - tags
  except:
    variables:
      - $SKIP_METRICS == "1"
```

Deploy jobs are disabled by default, you have to set in Gitlab UI a variable:

```bash
SKIP_DEPLOY 0
```

Then deploy jobs run by default on each push on `master` and `tag`. And they are
all set manual by default (must be manually started on the pipeline)

```yaml
.deploy_template:
#...
  only:
    refs:
      - master
      - tags
  except:
    variables:
      - $SKIP_DEPLOY == "1"
#...
  when: manual
```

## Workflow proposed

Workflow used with this project, based on Git branches or tags.

By default a new tag or branch _testing_ trigger the build, unit tests,
security, qa, lint, manual deploy to test.
A branch _master_ trigger qa, lint, manual deploy

You can adapt _only_ and _except_ for your own workflow, see
[Gitlab documentation](https://docs.gitlab.com/ee/ci/yaml/#only-and-except-simplified)
and section [Triggering pipeline](#triggering-pipeline)

Deploy jobs are disabled by default, you have to set in Gitlab UI a variable:

```bash
SKIP_DEPLOY 0
```

### Branch master

[![gitlab-pipeline-master](https://gitlab.com/mog33/gitlab-ci-drupal/uploads/449abbb2c59e217dc0999621511545e8/gitlab-pipeline-master.png)](https://gitlab.com/mog33/gitlab-ci-drupal/pipelines/73438508)

### Branch testing

[![gitlab-pipeline-testing](https://gitlab.com/mog33/gitlab-ci-drupal/uploads/5971202471b49e1477fdd4ef2cbf1eb6/gitlab-pipeline-testing.png)](https://gitlab.com/mog33/gitlab-ci-drupal/pipelines/73438670)

## Advanced usage

You can take a look in `.gitlab-ci.yml` for the text `[CI_TYPE] [DEPLOY] [TESTING]`
as a first step of editing to match your project.

I use [Robo.li](https://robo.li/) with this [RoboFile](.gitlab-ci/RoboFile.php)
for running composer, phpunit and some specific tasks.

### Nightwatch.js for Drupal 8

Since Drupal 8.6, [Nightwatch.js](https://www.drupal.org/docs/8/testing/javascript-testing-using-nightwatch) is included as a Javascript test framework.

For now it is not really ready to be used as a replacement for _functional
Javascript_, but soon...

The CI tests here include 2 patches to be able to upgrade to nightwatch 1.2 and
being able to install Drupal from a profile:

- [[Security] Update yarn packages to fix 19 vulnerabilities by updating nightwatch](https://drupal.org/node/3059356)
- [Support install profile and language code params in drupalInstall Nightwatch command](https://drupal.org/node/3017176)

There is a variable in this project that you can set in Gitlab to select the
tests Nightwatch will run:

```bash
  # Only my module tests if set a @tag
  NIGHTWATCH_TESTS    --tag my_module
  # All tests except core
  NIGHTWATCH_TESTS    --skiptags core
```

### Behat tests for Drupal 8

Tests for [Behat](http://behat.org) are executed from the `tests/` folder of the
project.

Copy this folder on the root of your project and adapt `tests/features` to your
tests.

For Behat, Selenium is not needed thanks to the
[Behat Chrome extension.](https://gitlab.com/DMore/behat-chrome-extension.git).

Html output of the Behat report is done thanks to
[Behat Html formatter plugin](https://github.com/dutchiexl/BehatHtmlFormatterPlugin).

### PHPunit tests for Drupal 8

The pipeline in this project support Unit, Kernel, Functional,
[Functional Javascript](https://www.drupal.org/docs/8/phpunit/phpunit-javascript-testing-tutorial)
tests in Drupal 8, see
[Type of tests in Drupal 8](https://www.drupal.org/docs/8/testing/types-of-tests-in-drupal-8).

The location of tests is defined in [.gitlab-ci/phpunit.xml]. There is 2 set of
tests location:

- Custom modules and themes only
  - All `tests/` from `modules/custom/**` and `themes/custom/**`
- All tests from all folders

There is a Gitlab variable to select the tests:

```bash
PHPUNIT_TESTS     custom
```

Set this variable empty to run all tests.

### Codecov.io support in PHPUNIT Code coverage

Code coverage job support [Codecov.io](https://codecov.io/).

After creating an account on [Codecov.io](https://codecov.io/), create from the
Gitlab UI _> Settings > CI / CD > Variables_ a variable `CODECOV_TOKEN` with
your token value.

### Rules for linting / Code standards / QA

All rules match a [Drupal 8](https://www.drupal.org) project.

To adapt some rules, first look at `.gitlab-ci/.phpqa.yml`, `.gitlab-ci/.phpmd.xml`
and `.gitlab-ci/.sass-lint.yml`.

More options see:

- [Phpqa configuration](https://github.com/EdgedesignCZ/phpqa#advanced-configuration---phpqayml)
- [Phpqa .phpqa.yml](https://github.com/EdgedesignCZ/phpqa/blob/master/.phpqa.yml)

Eslint is based on the official
[Drupal 8 eslintrc.passing.json](https://git.drupalcode.org/project/drupal/raw/HEAD/core/.eslintrc.passing.json)

Stylelint is based on the official
[Drupal 8 stylelintrc.json](https://git.drupalcode.org/project/drupal/raw/HEAD/core/.stylelintrc.json)

[Sass-lint](.gitlab-ci/.sass-lint.yml) is based on
[Wolox](https://github.com/Wolox/frontend-bootstrap/blob/master/.sass-lint.yml)

### Accessibility with Pa11y

Accessibility tests with [Pa11y](https://pa11y.org/), tests are defined in
[.gitlab-ci/pa11y-ci.json](.gitlab-ci/pa11y-ci.json)

For setting your urls to test, adapt the urls section:

```json
  "urls": [
    {
      "url": "http://localhost",
      "screenCapture": "pa11y-home.png"
    }
  ]
```

When a test failed, a screen capture is recorded in the reports.

## Jobs detail

Available stages on the pipelines are:

```bash
- build
- tests
# On each push.
- code quality
- code lint
# Only on tag, when released.
- php code metrics
# [DEPLOY] Skipped by default, see SKIP_DEPLOY in .gitlab-ci-variables.yml
- deploy to testing
- deploy to staging
- deploy to production
```

Available jobs

| Name | Detail | Report  |
|---|---|:---:|
| Build | If a project : `composer install`, can be used to add build steps (composer run-script, grunt, webpack, yarn...) | No |
| Unit and kernel | Phpunit unit and kernel tests | xml and html |
| Code coverage | Phpunit unit and kernel tests generating coverage, Codecov.io support see [Codecov.io support](#codecovio-support) | xml and html |
| Functional | Phpunit functional test (Browser based tests) | xml and html |
| Functional Js | Phpunit functional javascript test (Browser with javascript based tests) | xml and html |
| Nightwatch Js | Nightwatch.js javascript test (Browser with javascript based tests), see [Nightwatch.js for Drupal 8](#nightwatchjs-for-drupal-8) | text and html |
| Security report | Symfony security-checker, look at versions in composer.lock | text |
| Behat tests | Support Behat tests from `tests` folder, see [Behat tests for Drupal 8](#behat-tests-for-drupal-8) | html |
| Pa11y | Accessibility tests with [Pa11y](https://pa11y.org/), tests are defined in [.gitlab-ci/pa11y-ci.json](.gitlab-ci/pa11y-ci.json) | text |
| Code quality | Code sniffer with _Drupal standards_ | html |
| Best practices | Code sniffer with _Drupal Best practices standard_ | html |
| Js lint | Javascript check with eslint (as used in Drupal core, with Drupal rules) | html |
| Css lint | Css check with stylelint (as used in Drupal core, with Drupal rules) | text |
| Sass lint | Sass check with sass-lint | html |
| Php metrics | Code metrics in a nice html report with phpmetrics | html |
| Php stats | Code stats with phploc, pdepend | html |
| Deploy to ... | Sample of deploy jobs with ssh to a host | No |

## CI image including tools

All tools are included in a specific [docker image](https://gitlab.com/mog33/drupal8ci).

Nothing could be done without a bunch of awesome humans building awesome tools.

- [Robo](https://robo.li)
- [Eslint](https://eslint.org/)
- [Sass-lint](https://github.com/sasstools/sass-lint)
- [Stylelint](https://github.com/stylelint/stylelint)
- [Nightwatch.js](https://www.drupal.org/docs/8/testing/javascript-testing-using-nightwatch)
- [PHPunit](https://phpunit.de)
- [Security-checker](https://github.com/sensiolabs/security-checker)
- [Behat](http://behat.org/en/latest)
- [Pa11y](https://pa11y.org)

Code quality is done using the wonderful Phpqa, a tool that integrate other Php
quality and analysis tools:

- [Phpqa](https://github.com/EdgedesignCZ/phpqa)
  - [Phploc](https://github.com/sebastianbergmann/phploc)
  - [Phpcpd](https://github.com/sebastianbergmann/phpcpd)
  - [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer)
  - [Phpmd](https://github.com/phpmd/phpmd)
  - [PHP-Parallel-Lint](https://github.com/JakubOnderka/PHP-Parallel-Lint)
  - [Pdepend](https://pdepend.org/)
  - [Phpmetrics](https://www.phpmetrics.org)

## Running the jobs locally with Docker

You can perform most of the tests locally (on `*Unix`) without installing any
tool or Drupal code using included [docker-compose.yml](tests/docker-compose.yml)
file in this project, require:

- [Docker engine 18+](https://docs.docker.com/install)
- [Docker compose 1.24+](https://docs.docker.com/compose/install)

Default scenario is to have only your custom code modules / themes in a `/web`
folder, then run:

```bash
docker-compose -f tests/docker-compose.yml up -d
```

If you include your own `composer.json` file for_Drupal_, you must edit the
[docker-compose.yml](tests/docker-compose.yml) to use an image without Drupal
and [.docker-compose.env](tests/.docker-compose.env).

An helper bash script can help you run the tests using docker, this is a copy
of the jobs from the [.gitlab-ci.yml](.gitlab-ci.yml) file.

The script will copy configuration files from this folder and ensure folders to
run the tests properly.

```bash
tests/run-tests-ci-locally.sh all
```

## Future plan

You want to help me make this better? Good! just PR!

I would like to:

- Add Nightwatch [visual regression testing](https://github.com/Crunch-io/nightwatch-vrt)

- Add a Drupal dev version so you can test your module for the next version

- Add a matrix option like Travis to test against multiple Php versions and
databases when [Gitlab-ci support it](https://gitlab.com/gitlab-org/gitlab-ce/issues/49557)

- Test if all of this is working with some distributions like Lightning or
Varbase

## Credits

A lot of help and inspiration from those wonderful projects:

- [https://github.com/AcroMedia/commerce-demo](https://github.com/AcroMedia/commerce-demo)
- [https://github.com/Lullabot/drupal8ci](https://github.com/Lullabot/drupal8ci)
- [https://gitlab.com/Lullabot/d8cidemo/tree/gitlab](https://gitlab.com/Lullabot/d8cidemo/tree/gitlab)
- [https://github.com/manumilou/gitlab-ci-example-drupal](https://github.com/manumilou/gitlab-ci-example-drupal)
- [https://bitbucket.org/mediacurrent/ci-tests](https://bitbucket.org/mediacurrent/ci-tests)

----

Want some help implementing this on your project? I provide Drupal 8 expertise
as a freelance, just [contact me](https://developpeur-drupal.com/en).
