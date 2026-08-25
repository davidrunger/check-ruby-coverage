# Check Ruby Coverage

Check Ruby Coverage is a GitHub Action that fails unless a Cobertura XML report shows 100% line coverage. When coverage is incomplete, the action writes a sorted list of uncovered `path:line` locations to both the workflow log and the job summary.

## Requirements

### Action runtime

The action deliberately does not install Ruby or gems. Before invoking it, the workflow must provide:

- A `ruby` executable on `PATH`. [`ruby/setup-ruby`](https://github.com/ruby/setup-ruby) is the usual way to install and select Ruby in a GitHub Actions job.
- The [`rexml`](https://github.com/ruby/rexml) gem, so that `require 'rexml/document'` succeeds. `simplecov-cobertura` depends on `rexml`, so projects configured as described below already satisfy this requirement after Bundler installs their bundle.

A composite action (which this is) runs in the environment of its containing job. Ruby configured by an earlier workflow step is therefore available to Check Ruby Coverage. The action does not run `ruby/setup-ruby`, install a bundle, or choose a Ruby version itself.

### Coverage report

The action reads a [Cobertura](https://cobertura.github.io/cobertura/) XML report. Ruby projects can generate that report with [`simplecov-cobertura`](https://github.com/dashingrocket/simplecov-cobertura):

```ruby
# Gemfile
group :test do
  gem 'simplecov'
  gem 'simplecov-cobertura', require: false
end
```

Configure SimpleCov before loading application code:

```ruby
# spec/spec_helper.rb
require 'simplecov'

if ENV['CI']
  require 'simplecov-cobertura'
  SimpleCov.formatter = SimpleCov::Formatter::CoberturaFormatter
  SimpleCov.coverage_dir('tmp/simple_cov')
end

SimpleCov.start
```

This configuration writes the default report path, `tmp/simple_cov/coverage.xml`. A project can use a different location and pass it through the action's `coverage-file` input.

The action checks an existing report; it does not run tests, configure SimpleCov, or collate coverage from multiple test processes. A parallel test suite must collate its results into one Cobertura report before invoking the action.

## Usage

Set up Ruby, install the bundle, run the tests that generate the report, and then invoke the action:

```yaml
steps:
  - name: Check out code
    uses: actions/checkout@v7

  - name: Set up Ruby
    uses: ruby/setup-ruby@v1
    with:
      bundler-cache: true

  - name: Run tests
    run: bin/rspec

  - name: Check code coverage
    uses: davidrunger/check-ruby-coverage@v1
```

Pin actions to full commit SHAs in production workflows when possible.

To check a report at another path:

```yaml
- name: Check code coverage
  uses: davidrunger/check-ruby-coverage@v1
  with:
    coverage-file: coverage/coverage.xml
```

## Behavior

Check Ruby Coverage:

- Fails if the report is missing or malformed.
- Fails if the report contains zero valid lines or invalid line totals.
- Fails unless every valid line is covered.
- Prints sorted, unique `path:line` locations for uncovered lines.
- Writes failure details and uncovered locations to the job summary.
- Produces no job summary when coverage is complete.

The action checks line coverage only. It does not enforce branch coverage.

## Inputs

### `coverage-file`

The Cobertura XML report to check. The default is `tmp/simple_cov/coverage.xml`.

## Development

Run the dependency-free test script with:

```sh
ruby test/check-ruby-coverage
ruby test/release
```

The test harness adds no development dependencies. Running it requires the same Ruby and `rexml` prerequisites as running the checker itself.

See [RELEASING.md](RELEASING.md) for the automated release process. Release history is recorded in [CHANGELOG.md](CHANGELOG.md).

## License

Check Ruby Coverage is available under the [MIT License](LICENSE).
