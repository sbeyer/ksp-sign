KSP-Sign
========
*... a signing tool for lazy people after keysigning parties*

(C) 2005-2014, Stephan Beyer

> License: GNU GPL2,
> also means: NO WARRANTY!

Requirements
------------

* UNIX-like OS
* GnuPG (only tested with 1.4.14)
* Ruby 1.9+ interpreter
* and either: a running connection to the internet and /usr/sbin/sendmail
* or: a mail user agent capable of reading mbox files

Recommended
-----------

* Ruby-Termios
  (`apt-get install ruby-termios`
   or get it from <http://arika.org/ruby/termios>)


What is a key signing party? / References
------------------------------------------

You do not need ksp-sign, if you do not know what a key signing party (KSP)
is, or if you've never taken part or do not intend to take part in a key
signing party. Nevertheless you should read:

* The GnuPG Keysigning Party Howto:
          [en](http://www.cryptnet.net/fdp/crypto/gpg-party.html)
          [de](http://alfie.ist.org/projects/gpg-party/gpg-party.de.html)
          [es](http://members.fortunecity.com/keyparty/gpg-party.es.html)
          [it](http://www.gnupg.org/howtos/it/gpg-party.html)
          [ru](http://ivlad.unixgods.net/gpg-party/gpg-party-howto-ru.html)
          [si](http://neonatus.net/~neonatus/GPG/gpg-party-howto-si.html)
          [zh-tw](http://www.zope.org.tw/Members/pwchi/Tech_Docs/pgp-party)

For general information on GnuPG see:

* some self-advertising; my German article about GnuPG:
          [static](http://www.pro-linux.de/berichte/gnupg.html)
          [wiki](http://de.wikibooks.org/wiki/GnuPG)
* [official documentation](http://www.gnupg.org/documentation/index.html)
* [The GNU Privacy Handbook](http://www.gnupg.org/gph/en/manual.html)


What does the ksp-sign tool do?
-------------------------------

Have you ever signed more than 60 people after a key signing party?
Annoying? Indeed, there are dozens of Perl and Bash scripts, but either
they are too complex and/or they still leave too much work to the user.

(Note, I have written this in 2005. Now, as of 2014, there is perhaps
 much more useful and less buggy software. I have not checked.)


Usage
------

Most KSP coordinators or KSP web tools (like biglumber.com) generate a KSP
key ring containing all the keys for the KSP. KSP-Sign reads this key ring,
signs all uids and sends encrypted mails (the uid signature is attached) to
the owners of the keys. It is also capable to deal with photo ids and UIDs
without an e-mail address [but there's no special way to handle JabberIDs -
those are recognized as e-mail addresses]. The recipient does not need to
send a reply mail to you. If he received the mail and was able to decrypt
it, everything is ok. It's up to him to import the signature and put it on
a keyserver.

The ksp-sign tool uses your default gpg.conf file, it doesn't change
`GNUPGHOME` or set `--homedir`. You don't need to import the keys into your
pubring. KSP-Sign will use temporary key rings. You don't need to copy your
secring to another directory. Also, your trustdb isn't touched: a temporary
trustdb is used. But because it's faster, the option
`--no-auto-check-trustdb` is always added, so please ignore the GnuPG
`--check-trustdb` warning.


### Configuration

Take the file `ksp-sign.config.example` and make a copy of it named
`ksp-sign.config.rb`. Take your favourite editor (Ruby syntax highlighting
is nice for editing) and edit the `ksp-sign.config.rb` file. The top of that
file contains the *basic configuration*. Because ksp-sign is able to send
mails, you should at least edit the `$from` and (if you choose my
suggestion) the `event` variable.
You can also remove the line with the `event` variable and edit `$subject`
and `$text` yourself. You can and should use special `%variables%` - they're
introduced in the comments above. I think that's not too hard to configure.

Next there is the *sign-only feature*: We cannot send encrypted mails to
users with a sign-only key. But the encryption is important to ensure that
the owner really owns the private key. To get this information from a
sign-only key owner, we want him to reply to our mail. The reply must be
signed by that sign-only key! We also generate a random string the owner
has to send back in his reply. KSP-Sign wants to make the signature process
as easy as possible for you, but it's not an e-mail bot to get the reply
and send the signature. But it still makes it easy for you: look into the
`~/.ksp-sign/signonly/` directory. There are `key*.txt` files containing the
string that you'll have to compare, and `key*.asc` files with the signature
that you'll have to attach or upload to a keyserver.

This is the recommended behaviour for sign-only keys. But you can switch it
off by setting `$sign_only_secure` to `false`. KSP-Sign will send
unencrypted mails with the signature attached then, which is not
recommended!
The `$sign_only_text` variable contains the mail text being sent, giving
instructions how the verification process works. It will only be used if
`$sign_only_secure` is `true`.

Last but not least there are two booleans to set: A true `$refreshkeyring`
lets KSP-Sign update the keyring from your standard keyserver. The update
process may take some time and isn't always useful, so I usually choose
`false`.
The `$sendmail` variable specifies whether you want if KSP-Sign should send
mails (using /usr/sbin/sendmail) or not. If you have never used KSP-Sign I
recommend to start with `false`. Then KSP-Sign generates an mutt-readable
mbox file `~/.ksp-sign/mbox`.
There you can see how the generated mails look like.

I considered using Maildir, but the script is written for me and I use mbox
;) So I could copy it to `~/Mail/postponed` and send the mails using mutt.
Or one can choose to use an mbox sending script like the one that can be
found as [Gist](https://gist.github.com/sbeyer/8230267).


### Invocation

To begin, start ksp-sign with

    ./ksp-sign keyringfile

If you have multiple keys and want to sign the keyring with them, use

    ./ksp-sign keyringfile keyid1 keyid2

Now KSP-Sign asks you for the passphrase(s).
WARNING1: If the Ruby-Termios library isn't installed, everyone looking
at the screen can see it!
WARNING2: The passphrase is put into insecure memory!

I guess you have to wait some seconds until you are shown key id,
fingerprint and newly signed uids. Here KSP-Sign asks you to confirm the
fingerprint with your KSP data sheet. You have to type `y` and RETURN and
mails are sent to each uid. Typing anything else will not send any mail.

KSP-Sign uses `~/.ksp-sign/` for temporary files, the generated mbox and the
sign-only feature ;) You may delete it after you've finished all the
signings.


Discussion
----------

I wrote KSP-Sign to sign most of the people after the Chemnitzer Linux-Tage
2005 in Germany (a great – small, though – free software event in Germany).

First I wanted to make an inofficial `gpg --ksp-sign` patch for GnuPG 1.4.
Not, because a tool like `--ksp-sign` should be part of GnuPG, but because
it should be able to use low-level functions on OpenPGP keyblocks. The
result would have been a fast and clean KSP signing tool which does not
depend on the input/output behaviour of a special GnuPG version. (Some
signing helper tools did a perfect job on GnuPG 1.2, but not on GnuPG 1.4.)

Well, I decided to make a script using a high-level programming language
(like Lisp, Perl, Python or Ruby) and a high-level GnuPG backend. This is
much slower, because it has to run GnuPG more than once and filter out the
important information. But it is much more flexible and you don't have to
patch your gpg source with buggy C code, you just have to use buggy Ruby
code instead :)

Please contact me, if you prefer the patch variant mentioned above ;)

A second topic: This script is not able to run under Windows (and other
non-UNIX-like systems, I guess). Why? First because, I cannot test it on
other systems than mine. I used some UNIX-specific stuff like `/` as a path
delimiter, like `~/` for the home directory, like `/usr/sbin/sendmail` as
an SMTP wrapper, like `2>&1 >/dev/null` or `command &` stuff. Feel free to
make a Windows port but please contact me, so that I can put the changes
into the original script :)


Bugs
----

KSP-Sign is unrobust (...unstable, unportable, ...) by definition, or let's
say: broken by design. But it works for me. :)

If *sending mail* hangs, you have to activate the `$IgnorePossibleBug`
variable some lines below the basic configuration part in the source. Then
KSP-Sign uses nasty and slow code to avoid blocking IO. It never happened
to me, so the priority was low to get a suitable solution.
Another solution could be to encrypt files and not on-the-fly.

Many other sigs are deleted, but not all. I know how to solve this problem,
but I was to lazy :)

KSP-Sign communicates with GnuPG. GnuPG sometimes asks questions and
KSP-Sign answers them automatically using a given set of question-
answer-pairs. If a question isn't known, please mail me the output of
KSP-Sign, and I try to fix it.

This is not a bug, but there is an untested *lazy mode* you can activate. I
STRONGLY RECOMMEND NOT TO USE LAZY MODE. EVEN THE AUTHOR DOESN'T USE IT (IN
MOST CASES). In lazy mode you are asked *nothing* except the passphrase and
mail is sent to *everyone* in the key ring. So before running ksp-sign you
have to delete all the *wrong* keys from the key ring and check the
fingerprints!!

Last but not least, KSP-Sign has not ever been tested with GnuPG 2.


And now?
--------

Have fun!

Stephan Beyer <s-beyer@gmx.net>
