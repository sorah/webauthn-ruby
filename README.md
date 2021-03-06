# WebAuthn :key:

Easily implement WebAuthn in your ruby/rails app

[![Gem](https://img.shields.io/gem/v/webauthn.svg?style=flat-square)](https://rubygems.org/gems/webauthn)
[![Travis](https://img.shields.io/travis/cedarcode/webauthn-ruby/master.svg?style=flat-square)](https://travis-ci.org/cedarcode/webauthn-ruby)

## What is WebAuthn?

- [WebAuthn article with Google IO 2018 talk](https://developers.google.com/web/updates/2018/05/webauthn)
- [Web Authentication API draft article by Mozilla](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API)
- [WebAuthn W3C Candidate Recommendation](https://www.w3.org/TR/webauthn/)
- [WebAuthn W3C Editor's Draft](https://w3c.github.io/webauthn/)

## Prerequisites

This gem will help your ruby server act as a conforming [_Relying-Party_](https://www.w3.org/TR/webauthn/#relying-party), in WebAuthn terminology. But for the [_Registration_](https://www.w3.org/TR/webauthn/#registration) and [_Authentication_](https://www.w3.org/TR/webauthn/#authentication) ceremonies to work, you will also need

### A conforming User Agent

Currently supporting [Web Authentication API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API):
  - [Mozilla Firefox](https://www.mozilla.org/firefox/) 60+
  - [Google Chrome](https://www.google.com/chrome/) 67+
  - Google Chrome 65 & 66 (Disabled by default, go to chrome://flags to enable Web Authentication API feature)

### A conforming Authenticator

- [Security Key by Yubico](https://www.yubico.com/product/security-key-by-yubico/) (used to test/develop this gem)

NOTE: Firefox states ([Firefox 60 release notes](https://www.mozilla.org/en-US/firefox/60.0/releasenotes/)) they only support USB FIDO2 or FIDO U2F enabled devices in their current implementation (version 60).
  It's up to the gem's user to verify user agent compatibility if any other device wants to be used as the authenticator component.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'webauthn'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install webauthn

## Usage

### Registration

#### Initiation phase

```ruby
credential_creation_options = WebAuthn.credential_creation_options

# Store the newly generated challenge somewhere so you can have it
# for the validation phase.
#
# You can read it from the resulting options:
credential_creation_options[:challenge]

# Send `credential_creation_options` to the browser, so that they can be used
# to call `navigator.credentials.create({ "publicKey": credentialCreationOptions })`
```

#### Validation phase

```ruby
attestation_object = "..." # As returned by `navigator.credentials.create`
client_data_json = "..." # As returned by `navigator.credentials.create`

attestation_response = WebAuthn::AuthenticatorAttestationResponse.new(
  attestation_object: attestation_object,
  client_data_json: client_data_json
)

# This value needs to match `window.location.origin` evaluated by
# the User Agent as part of the validation phase.
original_origin = "https://www.example.com"

if attestation_response.valid?(original_challenge, original_origin)
  # 1. Register the new user and
  # 2. Keep Credential ID and Credential Public Key under storage
  #    for future authentications
  #    Access by invoking:
  #      `attestation_response.credential.id`
  #      `attestation_response.credential.public_key`
else
  # Handle error
end
```

### Authentication

#### Initiation phase

Assuming you have the previously stored Credential ID, now in variable `credential_id`

```ruby
credential_request_options = WebAuthn.credential_request_options
credential_request_options[:allowCredentials] << { id: credential_id, type: "public-key" }

# Store the newly generated challenge somewhere so you can have it
# for the validation phase.
#
# You can read it from the resulting options:
credential_request_options[:challenge]

# Send `credential_request_options` to the browser, so that they can be used
# to call `navigator.credentials.get({ "publicKey": credentialRequestOptions })`
```

#### Validation phase

Assuming you have the previously stored Credential Public Key, now in variable `credential_public_key`

```ruby
authenticator_data = "..." # As returned by `navigator.credentials.get`
client_data_json = "..." # As returned by `navigator.credentials.get`
signature = "..." # As returned by `navigator.credentials.get`

assertion_response = WebAuthn::AuthenticatorAssertionResponse.new(
  authenticator_data: authenticator_data,
  client_data_json: client_data_json,
  signature: signature
)

# This value needs to match `window.location.origin` evaluated by
# the User Agent as part of the validation phase.
original_origin = "https://www.example.com"

# This hash must have the id and its corresponding public key of the
# previously stored credential for the user that is attempting to sign in.
allowed_credential = {
  id: credential_id,
  publick_key: credential_public_key
}

if assertion_response.valid?(original_challenge, original_origin, allowed_credential: allowed_credential)
  # Sign in the user
else
  # Handle error
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake` to run the tests and code-style checks. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

### Commit message format

Each commit message follows the `<type>: <message>` format.

The "message" starts with lowercase and the "type" is one of:

* __build__: Changes that affect the build system or external dependencies
* __ci__: Changes to the CI configuration files and scripts
* __docs__: Documentation only changes
* __feat__: A new feature
* __fix__: A bug fix
* __perf__: A code change that improves performance
* __refactor__: A code change that neither fixes a bug nor adds a feature
* __style__: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
* __test__: Adding missing tests or correcting existing tests

Inspired in a subset of [Angular's Commit Message Guidelines](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/cedarcode/webauthn-ruby.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
