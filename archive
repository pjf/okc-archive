#!/usr/bin/env perl
use 5.010;
use strict;
use warnings;
use autodie;
use WWW::Mechanize;
use Config::Tiny;
use FindBin qw($Bin);
use utf8::all;

use constant DEFAULT_ARCHIVE => "$Bin/messages";

@ARGV or die "Usage: $0 thread-url [archive-dir]\n";

my $thread      = shift;
my $archive_dir = shift || DEFAULT_ARCHIVE;

# Make sure archive exists

if (not -d $archive_dir) {
    say "Creating $archive_dir...";
    mkdir $archive_dir;
}

my $mech = WWW::Mechanize::OKC->new(
    autocheck => 1,
    stack_depth => 0,
);

# Let's make sure OKC returns code similar to what I'll see in
# Firefox.  (Also let's not be quite so obviously a bot...)

$mech->agent_alias('Linux Mozilla');

# Login...
$mech->login;

if ($thread eq '-a') {
    say "Finding threads...";
    my @threads = $mech->mailbox;

    foreach my $thread (@threads) {
        $mech->backup($thread);
    }
}
else {
    $mech->backup($thread);
}

### Extended Mechanize class for OKC

package WWW::Mechanize::OKC;

use parent 'WWW::Mechanize';
use FindBin qw($Bin);
use autodie;

sub login {
    my ($mech) = @_;

    # TODO: Maybe use XDG paths?
    my $config = Config::Tiny->read("$Bin/auth.ini");

    unless ($config->{auth}{username}) {
        say "Hey, you need a `$Bin/auth.ini` file that looks like this:\n";
        say "[auth]\n",
            "username=your_username\n",
            "password=your_password\n"
        ;
        exit 1;
    }

    # Login dance!

    say "Logging in...";

    $mech->get("https://www.okcupid.com/login");

    $mech->submit_form(
        form_id   => 'login_form',
        fields    => {
            username => $config->{auth}{username},
            password => $config->{auth}{password},
        },
    );

    return;
}

sub mailbox {
    my ($mech) = @_;

    my $page = 0;

    my @threads;

    # Ugh. Magic loops.
    while (1) {

        # Ugh. Magic numbers
        my $start = 1 + ($page++ * 30);

        # Ugh. Magic strings
        my $inbox = "http://www.okcupid.com/messages?low=$start&infiniscroll=1&folder=1";

        $mech->get($inbox);

        my $content = $mech->content;

        # Break out if content is empty!
        last if ($content =~ /^\s*$/);

        # Sweet! We have more mailbox. Parse it!

        while ($content =~ m{
            <a[^>]*href="/profile/(?<from>[^"?]+)[^"]*"[^>]*class="photo"
            .*?
            <a\s*class="open"\s*href="(?<url>/messages[^"]*)"
        }msxg) {
            my $from = $+{from};
            my $url  = $+{url};
            $url =~ s{&amp;}{&}g;

            say $from;

            push(@threads, $url);
        }
    }

    # Here! Have some threads!
    return @threads;
}

sub backup {
    my ($mech, $thread) = @_;

    # Switch to https if not already specified.
    $thread =~ s{http:}{https:};

    my ($thread_id) = ($thread =~ m{threadid=(\d+)});

    # Fetch the message thread specified
    $mech->get($thread);

    my $content = $mech->content;

    my ($remote_user) = ($content =~ m{Your conversation with ([^"]+)});

    $remote_user or die "Oh noes! I don't know who this convo is with...\n";

    open(my $fh, '>', File::Spec->catfile($archive_dir, "$remote_user.$thread_id"));

    # Double-lines of output? Such ugly!

    say       "=== Conversation with $remote_user ($thread_id) ===\n";
    say {$fh} "=== Conversation with $remote_user ($thread_id) ===\n";

    # We started off trying to use XPath, but OKC doens't exactly return
    # well-formed XML. Instead, we'll use hacks...

    while ($content =~ m{
        <a[^>]*href="/profile/(?<from>[^"?]+)[^"]*"[^>]*class="photo"
        .*?
        <div\s*class="message"[^>]*>(?<content>.*?)</div>\s*</li>
    }msgx) {

        my $from = $+{from};
        my $msg  = $+{content};

        # Convert HTML to plain text... mostly.

        $msg =~ s{<div class="message_body">}{\n}msg;
        $msg =~ s{<script>.*?</script>}{}msg;
        $msg =~ s{</?(?:br|div)[^>]*/?>}{\n}msg;
        $msg =~ s{</?span[^>]*>}{}msg;
        $msg =~ s{<a [^>]*>}{}msg;
        $msg =~ s{</a>}{}msg;
        $msg =~ s{^[ ]*}{}msg;

        # Print our message.

        say {$fh} "[$from]\n$msg";
        say {$fh} "-" x 70;
    }
}
