# okc-archive

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.1.0/dist/gittip.png)](https://www.gittip.com/pjf/)

This is a simple script which lets you scrape an OKC conversation.

## Usage

1. Make yourself an `auth.ini` file that looks like this:

        [auth]
        username=your_username
        password=your_password

2. Run either `./archive URL_TO_CONVERSATION` to archive a single
conversation, or `./archive -a` to archive *all your conversations*.

3. Celebrate! The `messages/` directory will contain your conversations!
They will be named after the user you're talking to,
and contain the thread ID from the URL.

## Future

The code could do with refactoring. And tests. And quality control.

## Dependencies

* Perl (5.10.0 or later)
* Config::Tiny
* WWW::Mechanize
* utf8::all

## LICENSE

Same as Perl 5 itself.
