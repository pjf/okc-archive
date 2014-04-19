# okc-archive

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.1.0/dist/gittip.png)](https://www.gittip.com/pjf/)

This is a simple script which lets you scrape an OKC conversation.

## Usage

1. Make yourself an `auth.ini` file that looks like this:

        [auth]
        username=your_username
        password=your_password

2. Run `./archive URL_TO_CONVERSATION`

3. Celebrate! The `messages/` directory will have a file with your
conversation it! It will be named after the user you're talking to,
and contain the thread ID from the URL.

## Future

This would be even better if it auto-detected all
your conversations and archived them automatically.

## Dependencies

* Perl (5.10.0 or later)
* Config::Tiny
* WWW::Mechanize

## LICENSE

Same as Perl 5 itself.
