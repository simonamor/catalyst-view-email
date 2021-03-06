Catalyst::View::Email - Send Email from Catalyst
------------------------------------------------

This module sends out emails from a stash key specified in the
configuration settings.

#### Configuration:

**WARNING:** since version 0.10 the configuration options slightly changed!

Use the helper to create your View:

    $ script/myapp_create.pl view Email Email

In your app configuration:

```perl
    __PACKAGE__->config(
        'View::Email' => {
            # Where to look in the stash for the email information.
            # 'email' is the default, so you don't have to specify it.
            stash_key => 'email',
            # Define the defaults for the mail
            default => {
                # Defines the default content type (mime type). Mandatory
                content_type => 'text/plain',
                # Defines the default charset for every MIME part with the 
                # content type text.
                # According to RFC2049 a MIME part without a charset should
                # be treated as US-ASCII by the mail client.
                # If the charset is not set it won't be set for all MIME parts
                # without an overridden one.
                # Default: none
                charset => 'utf-8'
            },
            # Setup how to send the email
            # all those options are passed directly to Email::Sender::Simple
            sender => {
                # if mailer doesn't start with Email::Sender::Simple::Transport::,
                # then this is prepended.
                mailer => 'SMTP',
                # mailer_args is passed directly into Email::Sender::Simple 
                mailer_args => {
                    host     => 'smtp.example.com', # defaults to localhost
                    sasl_username => 'sasl_username',
                    sasl_password => 'sasl_password',
            }
            }
        }
    );
```

**NOTE ON SMTP**: If you use SMTP and don't specify host, it will default
to localhost and attempt delivery. This often means an email will sit in a
queue and not be delivered.

#### Sending Email

Sending email is just filling the stash and forwarding to the view:

```perl
    sub controller : Private {
        my ( $self, $c ) = @_;

        $c->stash->{email} = {
            to      => 'jshirley@gmail.com',
            cc      => 'abraxxa@cpan.org',
            from    => 'no-reply@foobar.com',
            subject => 'I am a Catalyst generated email',
            body    => 'Body Body Body',
        };

        $c->forward( $c->view('Email') );
    }
```

Alternatively you can use a more raw interface and specify the headers
as an array reference like it is passed to Email::MIME::Creator. Note
that you may also mix both syntaxes if you like ours better but need to
specify additional header attributes. The attributes are appended to the
header array reference without overwriting contained ones.

```perl
    $c->stash->{email} = {
        header => [
            To      => 'jshirley@gmail.com',
            Cc      => 'abraxxa@cpan.org',
            Bcc     => join ',', qw/hidden@secret.com hidden2@foobar.com/,
            From    => 'no-reply@foobar.com',
            Subject => 'Note the capitalization differences',
        ],
        body => qq{Ain't got no body, and nobody cares.},
        # Or, send parts
        parts => [
            Email::MIME->create(
                attributes => {
                    content_type => 'text/plain',
                    disposition  => 'attachment',
                    charset      => 'US-ASCII',
                },
                body => qq{Got a body, but didn't get ahead.},
            )
        ],
    };
```

You can set the envelope sender and recipient as well:

```perl
      $c->stash->{email} = {

        envelope_from => 'envelope-from@example.com',
        from          => 'header-from@example.com',

        envelope_to   => [ 'foo@example.com', 'bar@example.com' ],
        to            => 'Undisclosed Recipients:;',

        ...
      };
```

#### Handling Errors

If the email fails to send, the view will die (throw an exception).
After your forward to the view, it is a good idea to check for errors:

```perl
    $c->forward( $c->view('Email') );

    if ( scalar( @{ $c->error } ) ) {
        $c->error(0); # Reset the error condition if you need to
        $c->response->body('Oh noes!');
    } else {
        $c->response->body('Email sent A-OK! (At least as far as we can tell)');
    }
```

#### Using Templates for Email

Now, it's no fun to just send out email using plain strings. Take a look
at Catalyst::View::Email::Template to see how you can use your favourite
template engine to render the mail body.

#### Methods

* **new** - Validates the base config and creates the Email::Sender::Simple
object for later use by process.

* **process($c)** - The process method does the actual processing when the
view is dispatched to. This method sets up the email parts and hands off to
Email::Sender::Simple to handle the actual email delivery.

* **setup_attributes($c, $attr)** - Merge attributes with the configured
defaults. You can override this method to return a structure to pass into
generate_message which subsequently passes the return value of this method
to Email::MIME->create under the "attributes" key.

* **generate_message($c, $attr)** - Generate a message part, which should
be an Email::MIME object and return it. Takes the attributes, merges with
the defaults as necessary and returns a message object.

#### Troubleshooting

As with most things computer related, things break. Email even more so.
Typically any errors are going to come from using SMTP as your sending
method, which means that if you are having trouble the first place to
look is at Email::Sender::Transport::SMTP. This module is just a wrapper
for Email::Sender::Simple, so if you get an error on sending, it is
likely from there anyway.

If you are using SMTP and have troubles sending, whether it is
authentication or a very bland "Can't send" message, make sure that you
have Net::SMTP and, if applicable, Net::SMTP::SSL installed.

It is very simple to check that you can connect via Net::SMTP, and if
you do have sending errors the first thing to do is to write a simple
script that attempts to connect. If it works, it is probably something
in your configuration so double check there. If it doesn't, well, keep
modifying the script and/or your mail server configuration until it
does!

#### See Also

* Catalyst::View::Email::Template - Send fancy template emails with Cat
* Catalyst::Manual - The Catalyst Manual
* Catalyst::Manual::Cookbook - The Catalyst Cookbook

#### Authors

J. Shirley <jshirley@gmail.com>

Alexander Hartmaier <abraxxa@cpan.org>

#### Contributors

(Thanks!)

Matt S Trout

Daniel Westermann-Clark

Simon Elliott <cpan@browsing.co.uk>

Roman Filippov

Lance Brown <lance@bearcircle.net>

Devin Austin <dhoss@cpan.org>

Chris Nehren <apeiron@cpan.org>

#### COPYRIGHT

Copyright (c) 2007 - 2015 the Catalyst::View::Email "AUTHORS" and
"CONTRIBUTORS" as listed above.

#### LICENSE

This library is free software, you can redistribute it and/or modify it
under the same terms as Perl itself.

