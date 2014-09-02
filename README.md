![Build Status](https://travis-ci.org/watu/mail_safe.svg?branch=master)

# mail_safe

Mail safe provides a safety net while you're developing an application that uses ActionMailer.
It keeps emails from escaping into the wild.

Once you've installed and configured this gem, you can rest assure that your app won't send
emails to external email addresses.  Instead, emails that would normally be delivered to external
addresses will be sent to an address of your choosing, and the body of the email will be appended
with a note stating where the email was originally intended to go.

## Download

Github: http://github.com/myronmarston/mail_safe

Gem: `gem install mail_safe`

## Installation

Load the gem in the appropriate environments using Rails' 3.2+ gem support.  For example, I'm loading this in Gemfile as:

  `gem "mail_safe", group: [:development, :staging]`

IMPORTANT: Be sure not to load this in your production environment, otherwise, your emails won't be sent to the proper
recipients. In your test environment, you probably won't want this either. Rails ensures that no emails are ever sent in the
test environment, and tests that check outbound email recipients may fail.

## Configuration

In many cases, no configuration is necessary.  If you have git installed, and you've registered your email address
with it (check with "git config user.email" in your shell), mail safe will use this.  All emails will be sent to this address.

Otherwise, you can configure mail safe's behavior.  Create a file at config/initializers/mail_safe.rb, similar to the following:

```ruby
  if defined?(MailSafe::Config)
    MailSafe::Config.internal_address_definition = /.*@my-domain\.com/i
    MailSafe::Config.replacement_address = 'me@my-domain.com'
  end
```

The internal address definition determines which addresses will be ignored (i.e. sent normally) and which will be replaced.  Email
being sent to internal addresses will be sent normally; all other email addresses will be replaced by the replacement address.

These settings can also take procs if you need something more flexible:

```ruby
  if defined?(MailSafe::Config)
    MailSafe::Config.internal_address_definition = lambda { |address|
      address =~ /.*@domain1\.com/i ||
      address =~ /.*@domain2\.com/i ||
      address == 'full-address@domain.com'
    }

    # Useful if your mail server allows + dynamic email addresses like gmail.
    MailSafe::Config.replacement_address = lambda { |address| "my-address+#{address.gsub(/[\w\-.]/, '_')}@gmail.com" }
  end
```

When mail safe replaces an email address, it appends a notice to the bottom of the email body, such as:

```
  **************************************************
  This email originally had different recipients,
  but MailSafe has prevented it from being sent to them.

  The original recipients were:
  - to:
   - external-address-1@domain.com
   - external-address-2@domain.com
  - cc:
   - external-address-3@domain.com

  **************************************************
```
## Version Compatibility and Continuous Integration

Tested with Travis using Ruby 1.9.3, 2.0.0 and 2.1.2 against actionmailer 3.2, 4.0 and 4.1. ![Build Status](https://travis-ci.org/watu/mail_safe.svg?branch=master)


## Copyright

Copyright (c) 2009-2010 Myron Marston, Kashless.org. See LICENSE for details.
