# GitHub Workflows for Adobe Commerce

This repository contains reusable CI/CD workflows for Adobe Commerce.

**Author:** Patryk Waluś (patryk.walus@vaimo.com)

## Supported versions

- v1

  - Initial version of the workflows.
  - Possibility to enable/disable compilation step in the CI workflow
  - Change the format of input variables
  - Codacy CI Project Workflow - enables code quality checks and uploading of Code Coverage Reports to Codacy.

| Magento Version | PHP Version | Composer Version | Mariadb Version | Search Engine     | RabbitMQ Version |
| --------------- | ----------- | ---------------- | --------------- | ----------------- | ---------------- |
| 2.4.3           | 7.4         | 2.1              | 10.2            | Elasticsearch 7.9 | 3.8              |
| 2.4.6           | 8.1         | 2.2              | 10.6            | OpenSearch 2.5    | 3.9              |

# Usage

In order to utilize one of the workflows, it is necessary to include this configuration within the .yml file of your project.

## Adobe Commerce - Project CI Workflow

This workflow is designed to run the following tasks for your project:

- Compilation
- Code Sniffer
- Mess Detector
- Unit Tests
- Integration Tests

You can customize the workflow by providing the following parameters:

- full-analysis - decide whether to perform a full analysis of your project or only a partial analysis of changed files in your PR.
- enable-integration-tests - if you don't have integration tests, you can disable this option to speed up the workflow. However, it is recommended to start implementing integration tests in your project.
- enable-compilation - if you don't need to compile your project, you can disable this option to speed up the workflow.

```yaml
jobs:
  # Name of the job
  ci-workflow:
    # Specify the workflow and it's version
    uses: vaimo/github-workflows-magento/.github/workflows/ci-project.yml@v1
    with:
      # Adobe Commerce version
      # Required
      version: 2.4.6
      # Directory name that will be used to run action.
      # Optional - if not specified then uses the root directory.
      working-directory: src
      # Select directories that should be evaluated by code sniffer (separated by comma).
      # Optional - if not specified then uses the app/code/Vaimo as default
      code-sniffer-directories: app/code/Vaimo,app/code/Diptyque
      # Select directories that should be evaluated by mess detector (separated by comma).
      # Optional - if not specified then uses the app/code/Vaimo as default
      mess-detector-directories: app/code/Vaimo,app/code/Diptyque
      # Unit test suites to be executed.
      # Optional - if not specified then runs all tests defined in a phpunit.xml
      unit-tests-suites: UnitTests
      # Integration test suites to be executed.
      # Optional - if not specified then runs all tests defined in a phpunit.xml
      integration-tests-suites: IntegrationTests
      # Perform full code-sniffer/mess-detector analysis if true, partial analysis if false
      # Optional - if not specified then uses the true as default
      full-analysis: true
      # Enable unit tests
      # Optional - if not specified then uses the true as default
      enable-unit-tests: true
      # Enable integration tests
      # Optional - if not specified then uses the true as default
      enable-integration-tests: true
      # Enable compilation
      # Optional - if not specified then uses the true as default
      enable-compilation: true
      # Enable mess detector
      # Optional - if not specified then uses the true as default
      enable-mess-detector: true
      # Enable code sniffer
      # Optional - if not specified then uses the true as default
      enable-code-sniffer: true
    secrets:
      composer-auth: ${{secrets.COMPOSER_AUTH}}
```

## Adobe Commerce - Project CI Workflow (With Codacy)

This workflow is designed to run the following tasks for your project:

- Compilation
- Unit Tests
- Integration Tests

If this workflow is used, code quality checks will be done by Codacy. Therefore, code-sniffer and mess-detector are removed from the workflow. Additionally, the code coverage report from unit tests will be uploaded to Codacy.

You can customize the workflow by providing the following parameters:

- enable-integration-tests - if you don't have integration tests, you can disable this option to speed up the workflow. However, it is recommended to start implementing integration tests in your project.
- enable-compilation - if you don't need to compile your project, you can disable this option to speed up the workflow.

```yaml
jobs:
  # Name of the job
  ci-workflow:
    # Specify the workflow and it's version
    uses: vaimo/github-workflows-magento/.github/workflows/ci-project-codacy.yml@v1
    with:
      # Adobe Commerce version
      # Required
      version: 2.4.6
      # Directory name that will be used to run action.
      # Optional - if not specified then uses the root directory.
      working-directory: src
      # Unit test suites to be executed.
      # Optional - if not specified then runs all tests defined in a phpunit.xml
      unit-tests-suites: UnitTests
      # Integration test suites to be executed.
      # Optional - if not specified then runs all tests defined in a phpunit.xml
      integration-tests-suites: IntegrationTests
      # Enable integration tests
      # Optional - if not specified then uses the true as default
      enable-integration-tests: true
      # Enable compilation
      # Optional - if not specified then uses the true as default
      enable-compilation: true
    secrets:
      composer-auth: ${{secrets.COMPOSER_AUTH}}
      codacy-token: ${{secrets.CODACY_PROJECT_TOKEN}}
```

## Adobe Commerce - Module CI Workflow

This workflow is designed to run the following tasks for your module:

- Compilation
- Code Sniffer
- Mess Detector
- Unit Tests

_Integration tests to come in the future._

```yaml
jobs:
  # Name of the job
  ci-workflow:
    # Specify the workflow and it's version
    uses: vaimo/github-workflows-magento/.github/workflows/ci-module.yml@v1
    with:
      # Adobe Commerce version
      # Required
      version: 2.4.6
      # Set module source directory. Only necessary for the modules with the src structure.
      # Optional - if not specified then uses the ./ as default
      module-source-directory: src
    secrets:
      composer-auth: ${{secrets.COMPOSER_AUTH}}
```

## Project Setup Checklist

### Git

Directories `setup` and `dev` must be included in GIT:

```bash
git add -f app/etc/config.php
```

### Code Quality

In this step, `phpcs.xml` and `phpmd.xml` files will be used as a configuration for `ci.workflow` and additionally as a source of rulesets for PHPStorm.

#### Create a file in `coding-standards/code-sniffer/phpcs.xml`

```xml
<ruleset name="" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="https://raw.githubusercontent.com/squizlabs/PHP_CodeSniffer/master/phpcs.xsd">
    <!-- A fully annotated list of options is available here: https://github.com/squizlabs/PHP_CodeSniffer/wiki/Annotated-Ruleset -->
    <!-- File extensions to check -->
    <arg name="extensions" value="php,phtml,graphqls"/>
    <!--  Strip base directory from file paths in output -->
    <arg name="basepath" value="."/>
    <!-- Use colored output -->
    <arg name="colors"/>
    <!-- Show progress bar -->
    <arg value="p"/>
    <!-- Show sniff message codes in errors -->
    <arg value="s"/>
    <!-- Configures overall minimum severity -->
    <arg name="severity" value="1"/>
    <!-- Configure minimum severity for errors -->
    <arg name="error-severity" value="1"/>
    <!-- Configure minimum severity for warnings -->
    <arg name="warning-severity" value="1"/>

    <!-- Files to check -->
    <file>./app/code/Vaimo</file>

    <!-- Files to exclude -->
    <exclude-pattern>*/Test/*</exclude-pattern>

    <rule ref="Vaimo"/>
</ruleset>
```

#### Create a file in `coding-standards/mess-detector/phpmd.xml`

```xml
<?xml version="1.0"?>
<ruleset name="phpmd"
         xmlns="http://pmd.sf.net/ruleset/1.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0 http://pmd.sf.net/ruleset_xml_schema.xsd"
         xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">
    <description>
        PHP Mess Detector configuration
    </description>

    <rule ref="rulesets/unusedcode.xml"/>
    <rule ref="rulesets/cleancode.xml"/>
    <rule ref="rulesets/controversial.xml"/>
    <rule ref="rulesets/naming.xml">
        <exclude name="ShortVariable"/>
    </rule>
    <rule ref="rulesets/design.xml"/>
    <rule ref="rulesets/codesize.xml"/>

    <!-- Override rules -->
    <rule name="ShortVariable"
          since="0.2"
          message="Avoid variables with short names like {0}. Configured minimum length is {1}."
          class="PHPMD\Rule\Naming\ShortVariable"
          externalInfoUrl="https://phpmd.org/rules/naming.html#shortvariable">
        <description>
            Detects when a field, local, or parameter has a very short name.
        </description>
        <priority>3</priority>
        <properties>
            <property name="minimum" description="Minimum length for a variable, property or parameter name" value="2"/>
            <property name="exceptions" description="Comma-separated list of exceptions" value=""/>
        </properties>
        <example>
            <![CDATA[
class Something {
    private $q = 15; /exampleTION - Field
    public static function main( array $as ) { /exampleTION - Formal
        $r = 20 + $this->q; /exampleTION - Local
        for (int $i = 0; $i < 10; $i++) { /example Violation (inside FOR)
            $r += $this->q;
        }
    }
}
            ]]>
        </example>
    </rule>
</ruleset>
```

### Unit Tests

#### Create PHPUnit file in `dev/tests/unit/ci.workflow.phpunit.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
-->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/9.3/phpunit.xsd"
         colors="true"
         columns="max"
         beStrictAboutTestsThatDoNotTestAnything="false"
         bootstrap="./framework/bootstrap.php">
    <testsuites>
        <testsuite name="UnitTests">
            <directory>../../../app/code/Vaimo/*/Test/Unit</directory>
        </testsuite>
    </testsuites>
    <php>
        <includePath>.</includePath>
        <ini name="memory_limit" value="-1"/>
        <ini name="date.timezone" value="Europe/Warsaw"/>
        <ini name="xdebug.max_nesting_level" value="200"/>
    </php>
    <listeners>
        <listener class="Magento\Framework\TestFramework\Unit\Listener\ReplaceObjectManager"/>
    </listeners>
</phpunit>
```

Add file to git if it was not added by default:

```bash
git add -f dev/tests/unit/ci.workflow.phpunit.xml
```

### Integration Tests

Create the following files:

#### `dev/tests/integration/ci.workflow.phpunit.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
-->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/9.3/phpunit.xsd"
         colors="true"
         columns="max"
         beStrictAboutTestsThatDoNotTestAnything="false"
         bootstrap="./framework/bootstrap.php"
         stderr="true"
         testSuiteLoaderClass="Magento\TestFramework\SuiteLoader"
         testSuiteLoaderFile="framework/Magento/TestFramework/SuiteLoader.php">
    <testsuites>
        <testsuite name="IntegrationTests">
            <directory suffix="Test.php">../../../app/code/Vaimo/*/Test/Integration</directory>
        </testsuite>
    </testsuites>
    <php>
        <includePath>.</includePath>
        <includePath>testsuite</includePath>
        <ini name="date.timezone" value="Europe/Warsaw"/>
        <ini name="xdebug.max_nesting_level" value="200"/>
        <!-- Local XML configuration file ('.dist' extension will be added, if the specified file doesn't exist) -->
        <const name="TESTS_INSTALL_CONFIG_FILE" value="etc/ci.workflow.install-config-mysql.php"/>
        <!-- Local XML post installation configuration file ('.dist' extension will be added, if the specified file doesn't exist) -->
        <const name="TESTS_POST_INSTALL_SETUP_COMMAND_CONFIG_FILE" value="etc/ci.workflow.post-install-setup-command-config.php"/>
        <!-- Local XML configuration file ('.dist' extension will be added, if the specified file doesn't exist) -->
        <const name="TESTS_GLOBAL_CONFIG_FILE" value="etc/ci.workflow.config-global.php"/>
        <!-- Semicolon-separated 'glob' patterns, that match global XML configuration files -->
        <const name="TESTS_GLOBAL_CONFIG_DIR" value="../../../app/etc"/>
        <!-- Whether to cleanup the application before running tests or not -->
        <const name="TESTS_CLEANUP" value="enabled"/>
        <!-- Memory usage and estimated leaks thresholds -->
        <const name="TESTS_MEM_USAGE_LIMIT" value="4096M"/>
        <const name="TESTS_MEM_LEAK_LIMIT" value=""/>
        <!-- Path to Percona Toolkit bin directory -->
        <!--<const name="PERCONA_TOOLKIT_BIN_DIR" value=""/>-->
        <!-- CSV Profiler Output file -->
        <!--<const name="TESTS_PROFILER_FILE" value="profiler.csv"/>-->
        <!-- Bamboo compatible CSV Profiler Output file name -->
        <!--<const name="TESTS_BAMBOO_PROFILER_FILE" value="profiler.csv"/>-->
        <!-- Metrics for Bamboo Profiler Output in PHP file that returns array -->
        <!--<const name="TESTS_BAMBOO_PROFILER_METRICS_FILE" value="../../build/profiler_metrics.php"/>-->
        <!-- Whether to output all CLI commands executed by the bootstrap and tests -->
        <const name="TESTS_EXTRA_VERBOSE_LOG" value="1"/>
        <!-- Magento mode for tests execution. Possible values are "default", "developer" and "production". -->
        <const name="TESTS_MAGENTO_MODE" value="developer"/>
        <!-- Minimum error log level to listen for. Possible values: -1 ignore all errors, and level constants form http://tools.ietf.org/html/rfc5424 standard -->
        <const name="TESTS_ERROR_LOG_LISTENER_LEVEL" value="-1"/>
        <!-- Connection parameters for MongoDB library tests -->
        <!--<const name="MONGODB_CONNECTION_STRING" value="mongodb://localhost:27017"/>-->
        <!--<const name="MONGODB_DATABASE_NAME" value="magento_integration_tests"/>-->
        <!-- Connection parameters for RabbitMQ tests -->
        <!--<const name="RABBITMQ_MANAGEMENT_PROTOCOL" value="https"/>-->
        <!--<const name="RABBITMQ_MANAGEMENT_PORT" value="15672"/>-->
        <!--<const name="RABBITMQ_VIRTUALHOST" value="/"/>-->
        <!--<const name="TESTS_PARALLEL_RUN" value="1"/>-->
        <const name="USE_OVERRIDE_CONFIG" value="enabled"/>
    </php>
    <!-- Test listeners -->
    <listeners>
        <!-- Run after AllureAdapter to allow it to initialize properly -->
        <listener class="Magento\TestFramework\Event\PhpUnit"/>
        <listener class="Magento\TestFramework\ErrorLog\Listener"/>
    </listeners>
</phpunit>
```

#### `dev/tests/integration/etc/ci.workflow.config-global.php`

```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */

return [
    'customer/password/limit_password_reset_requests_method' => 0,
    'admin/security/admin_account_sharing' => 1,
    'admin/security/limit_password_reset_requests_method' => 0,
];
```

#### `dev/tests/integration/etc/ci.workflow.install-config-mysql.php`

```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */

return [
    'db-host' => 'mariadb',
    'db-user' => 'root',
    'db-password' => 'password',
    'db-name' => 'magento_integration_tests',
    'backend-frontname' => 'backend',
    'search-engine' => 'elasticsearch7',
    'elasticsearch-host' => 'elasticsearch',
    'elasticsearch-port' => '9200',
    'admin-user' => \Magento\TestFramework\Bootstrap::ADMIN_NAME,
    'admin-password' => \Magento\TestFramework\Bootstrap::ADMIN_PASSWORD,
    'admin-email' => \Magento\TestFramework\Bootstrap::ADMIN_EMAIL,
    'admin-firstname' => \Magento\TestFramework\Bootstrap::ADMIN_FIRSTNAME,
    'admin-lastname' => \Magento\TestFramework\Bootstrap::ADMIN_LASTNAME,
    'amqp-host' => 'rabbitmq',
    'amqp-port' => '5672',
    'amqp-user' => 'guest',
    'amqp-password' => 'guest',
    'disable-modules'   => join(',', []),
];
```

#### `dev/tests/integration/etc/ci.workflow.post-install-setup-command-config.php`

```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */

return [
    /tests [
        'command' => 'setup:config:set',
        'config' => [
            '--remote-storage-driver' => 'aws-s3',
            '--remote-storage-bucket' => 'myBucket',
            '--remote-storage-region' => 'us-east-1',
        ]
    ]
    */
];
```

Add files to git:

```bash
git add -f dev/tests/integration/ci.workflow.phpunit.xml dev/tests/integration/etc/ci.workflow.install-config-mysql.php dev/tests/integration/etc/ci.workflow.config-global.php dev/tests/integration/etc/ci.workflow.post-install-setup-command-config.php
```

### Github Workflow

Create a file in `.github/workflows/ci.yml` as described in the section **Usage** in this readme file.

### Github configuration

In the project repository settings, a COMPOSER_AUTH secret needs to be created. It needs to be passed a string. Use the following code as an example.

```text
"{\"http-basic\": {\"repo.magento.com\": {\"username\": \"{{username}}\", \"password\": \"{{password}}\"}}}"
```

