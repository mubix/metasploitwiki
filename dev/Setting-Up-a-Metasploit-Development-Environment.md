# Metasploit Development Environment

<sup>*The shortlink to this wiki page is http://r-7.co/MSF-DEV*</sup>

This is a guide for setting up an environment for effectively **contributing to the Metasploit Framework**. If you just want to use Metasploit for legal, authorized hacking, we recommend instead you [download the Metasploit binary installer](http://metasploit.com/download), which will take care of all the dependencies and give you access to the open source Metasploit Framework, the free Metasploit Community edition, and an option to start the free trial for Metasploit Pro.

If you're using Kali Linux, Metasploit is already pre-installed for non-development purposes; just type `msfconsole` in the terminal to start Metasploit Framework, then type `go_pro` if you'd like to try Metasploit Pro or Metasploit Community.

If you want to develop on and [contribute] to Metasploit, read on! This guide should get you going on pretty much any Debian-based Linux system, but it is written for Kali Linux in particular, since many, many Metasploit Users are also Kali Linux users, and why spin up a different VM?

If you're familiar with Ubuntu or Xandros or any other Debian distro, you should be able to read along here and get it to work for you. If there are distro-specific gotchas you spot, please [let us know][issues]!

If you would like to sometimes develop Metasploit-Framework, and sometimes just use the Metasploit Community Edition which ships with Kali, you will want to likely create separate user accounts. You might be able to get away with different Gnome Terminal profiles, but you're not running out of UIDs, I promise. At the very least, you're going to need a non-root account for Metasploit Framework development work.

For this guide, the example user is "YOUR_USERNAME," and the sample password in this document is "YOUR_PASSWORD." Anywhere you see those strings, use your own username and password. Obviously, they should be hard.

Each section will have a **TLDR** code snippet, suitable for copy-pasting, if you just want to speed through things, then a more complete explination of what's going on with the TLDR broken down into more of a step-by-step. Keep in mind that as written, many of these can overwrite any local customization you might have, may have less secure defaults than you'd like, and other surprises. Use them only if you are impatient, have done this all before, and understand the risks.

At the end of this document, there's a [TLDR of TLDRs](#tldr-of-tldrs). You can't yet run it all at once and go off to lunch, but setup should now be only a few lightly edited copy-pastes away.

<sup>*TODO: Ansible!*</sup>

So let's get started!

## Update Kali Linux

#### TLDR (as root)

----
```bash
echo deb http://http.kali.org/kali kali main non-free contrib > /etc/apt/sources.list &&
echo deb-src http://http.kali.org/kali kali main non-free contrib >> /etc/apt/sources.list &&
echo deb http://security.kali.org/kali-security kali/updates main contrib non-free >> /etc/apt/sources.list &&
apt-get clean &&
rm -rf /var/lib/apt/lists;
apt-get update &&
apt-get -y --force-yes install kali-archive-keyring &&
apt-get update &&
apt-get -y upgrade
```
----

First, you need to know where all the Linux goodness lives. Your `/etc/apt/sources.list` should have these sources listed:

```
deb http://http.kali.org/kali kali main non-free contrib
deb-src http://http.kali.org/kali kali main non-free contrib
deb http://security.kali.org/kali-security kali/updates main contrib non-free
```

If you're missing any of these, add them. If you have a lot of extras, you are almost certain to cause conflicts. [Don't do that][kali-sources]. Once you're set with sources, clean out any cruft, get the latest Kali signing key, and go to town:


```
apt-get clean
rm -rf /var/lib/apt/lists
apt-get update
apt-get -y --force-yes install kali-archive-keyring
apt-get update
apt-get -y upgrade
```

## Enable remote access

#### TLDR (as root)

----
```bash
apt-get -y install ufw;
ufw enable &&
ufw allow 4444:4464/tcp &&
ufw allow 8080:8090/tcp &&
ufw allow ssh &&
service ssh start
```
----

Often, you need to have remote access back to your Kali machine; a typical use case is for reverse shells. You might also want to use ssh and scp to write code and copy files, from elsewhere -- this is especially useful if you're running Kali as a guest OS and don't want to install VMWare Tools.

```
apt-get -y install ufw
ufw enable
ufw allow 4444:4464/tcp # For default reverse shells
ufw allow 8080:8090/tcp # For browser exploits
ufw allow ssh && service ssh start # If you want to shell in from elsewhere
```

## Create a Dev User

#### TLDR (as root)

----
```bash
useradd -m msfdev &&
PASS=`tr -dc A-Za-z0-9_ < /dev/urandom | head -c8`;
echo ** RECORD THIS: Your msfdev Kali user password is $PASS ** &&
echo "msfdev:$PASS" | chpasswd &&
unset PASS &&
usermod -a -G sudo msfdev &&
chsh -s /bin/bash msfdev
```

**SWITCH TO THIS NON-ROOT USER NOW.**
----

You will want to create a non-root user. In this example, the user is `msfdev`. Neither Git nor RVM likes you to be root, since weird things can easily happen with your filesystem permissions.

```
useradd -m msfdev
passwd msfdev # Set a decent password, or use a script
usermod -a -G sudo msfdev
chsh -s /bin/bash msfdev
```

Once this is complete, switch to this user by logging out of `root` and logging back in as `msfdev`. While some steps down the line will still require sudoer access, you should resist the temptation to keep being root. You will invariably forget to switch and start getting mystery errors about unable to read critical resources that RVM and Git need.

## Install the base dev packages

#### TLDR (as msfdev)

----
```bash
echo 'YOUR_PASSWORD_FOR_KALI' | sudo -kS apt-get -y install \
  build-essential zlib1g zlib1g-dev \
  libxml2 libxml2-dev libxslt-dev locate \
  libreadline6-dev libcurl4-openssl-dev git-core \
  libssl-dev libyaml-dev openssl autoconf libtool \
  ncurses-dev bison curl wget xsel postgresql \
  postgresql-contrib libpq-dev \
  libapr1 libaprutil1 libsvn1 \
  libpcap-dev libsqlite3-dev
```
----

The TLDR here is all you should need to stage up Kali (or any other Debian-based distro) for a proper dev environment. Note, there's no Ruby or editor -- we'll get to those next.

## Install RVM

#### TLDR (as msfdev)

----
```bash
curl -sSL https://rvm.io/mpapis.asc | gpg --import - &&
curl -L https://get.rvm.io | bash -s stable --autolibs=enabled --ruby=2.1.6 &&
source $HOME/.rvm/scripts/rvm &&
gem install bundler &&
ruby -v && # See that it's 2.1.6
sudo gconftool-2 --direct --config-source xml:readwrite:/etc/gconf/gconf.xml.defaults \
  --type boolean --set /apps/gnome-terminal/profiles/Default/login_shell true
```
----

Kali, like most operating system distributions, does not ship with the latest Ruby with any predictable frequency. So, we'll use RVM, the Ruby Version Manager. You can read up on it here: https://rvm.io/, and discover it's pretty swell. Some people prefer rbenv, and those instructions [are here][rbenv]. For our purposes, though, we're going to stick to RVM.

First, you need the signing key for the RVM distribution:

```
curl -sSL https://rvm.io/mpapis.asc | gpg --import -
```

Next, get RVM itself:

```
curl -L https://get.rvm.io | bash -s stable --autolibs=enabled --ruby=2.1.6
```

This does pipe straight to bash, which can be a [sensitive issue][dont-pipe]. For the longer, safer way:

```
curl - rvm.sh -L https://get.rvm.io
cat rvm.sh # Read it and see it's all good
cat rvm.sh | bash -s stable --autolibs=enabled --ruby=2.1.6
```

Once that's done, fix your current terminal to use RVM's version of ruby:

```
source $HOME/.rvm/scripts/rvm
ruby -v # See that it's 2.1.6
```

And finally, install the `bundler` gem in order to get all the other gems you'll need:

```
gem install bundler
```

### Configure Gnome Terminal to use RVM

To always use RVM's version of ruby in Gnome Terminal, run the
following:

```
sudo gconftool-2 --direct --config-source xml:readwrite:/etc/gconf/gconf.xml.defaults \
 --type boolean --set /apps/gnome-terminal/profiles/Default/login_shell true
```

Or, you can navigate to Edit > Profiles >
Highlight Default > Edit > Title and Command > Check **[ ] Run command
as a login shell**. It looks like this:

[[/screens/kali-gnome-terminal.png]]

Finally, see that you're now running Ruby 2.1.6:

```
ruby -v
```

It should say `ruby 2.1.6p336`, unless there is a later version and this doc hasn't been updated yet.

## Install an Editor

#### TLDR (as msfdev)

----
```
sudo apt-get install vim-gnome -y &&
curl -Lo- https://bit.ly/janus-bootstrap | bash
```
----

I like gvim, and many others do, too. I also like [Janus](https://github.com/carlhuda/janus), a collection of plugins that supports a bunch of different languages, including Ruby. The TLDR above will install both of these. Again, if you don't like piping curl output straight to bash, do it your saner, slower way.

Many choices of editor exist, of course. An informal straw poll shows that many Metasploit developers use:

  - [Rubymine](https://www.jetbrains.com/ruby/), a few use
  - a few use [emacs](http://i.imgur.com/dljEqtL.gif)
  - and still others use [Sublime Text](http://www.sublimetext.com/) with some helpful [plugins](https://gist.github.com/kernelsmith/5308291).

For this setup, though, let's just say you're a vim person, and move on.










[contribute]:http://r-7.co/MSF-CONTRIB
[issues]:https://github.com/rapid7/metasploit-framework/issues
[ffmike-gist]: https://gist.github.com/ffmike/877447
[kali-sources]: http://docs.kali.org/general-use/kali-linux-sources-list-repositories
[gitconfig]:https://github.com/todb-r7/junkdrawer/blob/master/dotfiles/git-repos/gitconfig
[dont-pipe]:http://www.seancassidy.me/dont-pipe-to-your-shell.html
[rbenv]:https://github.com/sstephenson/rbenv#installation
[generating-ssh-keys]:https://help.github.com/articles/generating-ssh-keys/
[2fa]:https://help.github.com/articles/about-two-factor-authentication/
[forking]:https://help.github.com/articles/fork-a-repo/
[cloning]:https://help.github.com/articles/fork-a-repo/#step-2-create-a-local-clone-of-your-fork
[Bundler]:http://bundler.io/
[metasploit-committer]:https://github.com/rapid7/metasploit-framework/wiki/Committer-Rights
[git-horror]:http://mikegerwitz.com/papers/git-horror-story
[signing-howto]:https://github.com/rapid7/metasploit-framework/wiki/Committer-Keys#signing-howto
[landing-prs]:https://github.com/rapid7/metasploit-framework/wiki/Landing-Pull-Requests
[todb]:https://github.com/todb-r7
[gh-pr-refs]:https://help.github.com/articles/checking-out-pull-requests-locally/