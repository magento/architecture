# Using phpro/grumphp for local quality check


## Why?
As a developer, you need fast feedback if you working on a pull request. 
Currently, you get feedback from  Travis after hours or days. With GrumPHP, the code can be quickly checked on every commit.

## Benefits:
 - Quik feedback for developers 
-  Better commits (fewer `fix phpcs style` commits)

Sample Config:
```yml
parameters:
    bin_dir: "./vendor/bin"
    git_dir: "."
    hooks_dir: ~
    hooks_preset: local
    stop_on_failure: false
    ignore_unstaged_changes: false
    ascii:
        succeeded: ~
        failed: ~
    tasks:
        phpversion:
            project: '7.1'
        xmllint:
            ignore_patterns: ["src/dev"]
        yamllint:
            ignore_patterns: ["src/dev"]
        securitychecker:
            lockfile: ./src/composer.lock
            format: ~
            end_point: ~
            timeout: ~
            run_always: false
        git_blacklist:
            keywords:
            - "die("
            - "var_dump("
            - "print_r("
            - "var_export("
            - "exit;"
            whitelist_patterns:
            - /^src\/app\/(.*)/
            triggered_by: ['php']
        phpmnd:
            directory: ./src/app/code
            whitelist_patterns: []
            exclude: []
            exclude_name: []
            exclude_path: []
            extensions: []
            hint: false
            ignore_numbers: []
            ignore_strings: []
            strings: false
            triggered_by: ['php']
        phpcs:
            standard: "./src/dev/tests/static/framework/Magento/ruleset.xml"
            severity: ~
            error_severity: ~
            warning_severity: 6
            tab_width: ~
            report: summary
            report_width: ~
            whitelist_patterns:
            - /^src\/app\/code\/(.*)/
            encoding: ~
            ignore_patterns: []
            sniffs: []
            triggered_by: [php]
        phpunit:
            config_file: ./dev/tests/phpunit-no-coverage.xml
            testsuite: ~
            group: []
            always_execute: false
```
