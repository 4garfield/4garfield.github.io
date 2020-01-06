---
title: "Mac Development Environment Setup"
date: 2019-12-31 18:19:30 +0800
header:
  image: "/assets/images/headers/mac-development-setup-header.jpg"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com/photos/xrVDYZRGdw4)"
tags:
  - mac
  - brew
  - zsh
  - java
  - nodejs
  - php
  - ruby
toc: true
---

This guide is for how to set up the Mac for software development.

## Xcode

Download latest Xcode from Apple Store, then open up a new terminal and type the following command: `xcode-select --install`.

The Command Line Tools Package is a small self-contained package available for download separately from Xcode and that allows you to do command line development in OS X. It consists of two components: OS X SDK and command-line tools such as Clang, which are installed in `/usr/bin`.

## Homebrew

[Homebrew](https://brew.sh): The missing package manager for macOS.

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Check brew installation: `brew  --version`.

By default, brew will install the formulas under `/usr/local/Cellar`, and create soft link in the `/usr/local/bin`.

Configure homebrew origin(if you're in China network):

```sh
cd "$(brew --repo)"
git remote set-url origin git://mirrors.ustc.edu.cn/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

Configure Homebrew Bottles default remote repo origin:

```sh
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

Configure brew alias: `echo 'alias brewupd="brew update && brew upgrade && brew cleanup"' >> ~/.bash_profile`

[Homebrew Cask](https://github.com/Homebrew/homebrew-cask): A CLI workflow for the administration of Mac applications distributed as binaries.

```sh
brew tap caskroom/cask
```

Configure homebrew-cask default remote origin:

```sh
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
```

Install the essential tools & apps:

```sh
brew install tree

brew cask install google-chrome
brew cask install firefox
brew cask install cheatsheet
brew cask install cleanmymac
brew cask install alfred
brew cask install dash
brew cask install android-studio
brew cask install sublime-text
brew cask install visual-studio-code
```

## iTerm2 + zsh + oh-my-zsh

Install the beautiful terminal & shell replacement app:

```sh
# iTerm2
brew cask install iterm2

# zsh
brew install zsh

# oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Configure the zsh environment, open `~/.zshrc` to add below lines:

```sh
source ~/.bash_profile

export PATH=$HOME/bin:/usr/local/bin:/usr/local/sbin:$PATH
```

Install the Nerd fonts:

```sh
brew tap caskroom/fonts
brew cask install font-hack-nerd-font
```

Configure iTerm2 to use the font by going to: `iTerm2 -> Preferences -> Profiles -> Text -> Font -> Change Font`, select the font **Hack Regular Nerd Font Complete** and adjust the size if your want too. Also check the box for `Use a different font for non-ASCII text` and select the font again.

Download & import iTerm2 [material-design color theme](https://github.com/MartinSeeler/iterm2-material-design).

Clone the zsh theme powerlevel9k: `git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k`, then configure the theme in `~/.zshrc`:

```sh
ZSH_THEME="powerlevel9k/powerlevel9k"

POWERLEVEL9K_MODE='nerdfont-complete'
```

Then customize powerlevel9k in `~/.zshrc`:

```sh
{% raw %}
POWERLEVEL9K_STATUS_VERBOSE=false
POWERLEVEL9K_PROMPT_ON_NEWLINE=true
POWERLEVEL9K_SHORTEN_DIR_LENGTH=3
POWERLEVEL9K_MULTILINE_FIRST_PROMPT_PREFIX=""
POWERLEVEL9K_MULTILINE_LAST_PROMPT_PREFIX="%F{cyan}\uF460%F{073}\uF460%F{109}\uF460%f"
POWERLEVEL9K_OS_ICON_BACKGROUND="white"
POWERLEVEL9K_OS_ICON_FOREGROUND="black"
POWERLEVEL9K_TIME_FORMAT="%D{%H:%M:%S}"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon dir vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status load battery time)
{% endraw %}
```

Configure the zsh [plugins](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins):

* zsh-syntax-highlighting: `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`
* zsh-autosuggestions: `git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`
* autoupdate: `git clone https://github.com/TamCore/autoupdate-oh-my-zsh-plugins ~/.oh-my-zsh/custom/plugins/autoupdate`

Then enable these plugins in `~/.zshrc`:

```sh
plugins=(brew git npm colored-man-pages autoupdate zsh-autosuggestions zsh-syntax-highlighting)
```

## Browser

To enable Safari developer tools, select "Preference" -> "Advanced" -> "Show Develop menu in menu bar".

## Java

Install openjdk: `brew cask install adoptopenjdk`. Check the installed java home path: `/usr/libexec/java_home`.

Manually configure the `JAVA_HOME` variable in shell: `echo 'export JAVA_HOME="`/usr/libexec/java_home`"' >> ~/.bash_profile`.

Configure the JVM options:

```sh
echo 'export JAVA_OPTS="-Xms512m -Xmx1024m"' >> ~/.bash_profile
```

Install java IDEs:

```sh
brew cask install eclipse-jee
brew cask install intellij-idea
```

Install maven: `brew install maven`. The maven global configuration file default location: `/usr/local/Cellar/maven/{{version}}/libexec/conf/settings.xml`.

Create user configuration file as `~/.m2/settings.xml`, add below configuration for proxy if your network is slow:

```xml
</mirrors>
  <mirror>
    <id>ustc</id>
    <mirrorOf>central</mirrorOf>
    <name>USTC maven proxy</name>
    <url>https://maven.proxy.ustclug.org/maven2/</url>
  </mirror>
</mirrors>
```

## Node.js

Install through brew command: `brew install node`, then configure remote registry: `npm config set registry https://registry.npm.taobao.org`. Then update the npm version: `npm install -g npm`.

## Apache Httpd

To use different apache httpd server instead of default version in macOS, you must unload the built-in apache service first:

```sh
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

Install the latest version of httpd server: `brew install httpd`, start as daemon service: `brew services start httpd`, the default listen port is 8080.

## Dnsmasq

Dnsmasq provides network infrastructure for small networks: DNS, DHCP, router advertisement and network boot. Installation via: `brew install dnsmasq`.

To resolve `*.localhost` and `*.test` domains, edit the config file `$(brew —-prefix)/etc/dnsmasq.conf` with:

```conf
address=/.localhost/127.0.0.1
address=/.test/127.0.0.1
```

For local development, edit the config file to listen on default DNS name server port.

```conf
port=53
```

Start dnsmasq as a service so it automatically starts at login: `sudo brew services start dnsmasq`

Test the configuration: `dig @127.0.0.1 -q foobar.localhost`, the successful result will contains below output section:

```sh
;; ANSWER SECTION:
foobar.localhost.   0   IN   A   127.0.0.1
```

On most UNIX-like systems the `/etc/resolv.conf` file determines how DNS queries are made. It's better to add separate resolver files inside the `/etc/resolver/` directory.

```sh
sudo mkdir -v /etc/resolver
sudo bash -c "echo 'nameserver 127.0.0.1' > /etc/resolver/localhost"
sudo bash -c "echo 'nameserver 127.0.0.1' > /etc/resolver/test"
```

**Note:** the watcher of `/etc/resolver` folder only watch once the file exists. It won't refresh the DNS resolver if the file contents changed.

Check the DNS resolver configuration: `scutil --dns`. And we can test if it's really works:

```sh
ping -c 1 google.com # Make sure default DNS resolver still works
ping -c 1 foobar.localhost
ping -c 1 foobar.test
```

## MariaDB

Installation through: `brew install mariadb`, then start daemon service: `brew services start mariadb`.

To login, just type: `mysql -uroot`. By default, MySQL has no root password, need set by following command: `mysqladmin -u root password 'password'`, then login using this password: `mysql -u root -p`

## PHP

Installation by: `brew install php`, then start as service: `brew services start php`.

To enable PHP in Apache httpd server, add the following to `httpd.conf` and restart Apache:

```apache
LoadModule php7_module /usr/local/opt/php/lib/httpd/modules/libphp7.so

<FilesMatch \.php$>
  SetHandler application/x-httpd-php
</FilesMatch>
```

Then, check the "DirectoryIndex" includes `index.php`: `DirectoryIndex index.php index.html`.

The `php.ini` and `php-fpm.ini` file can be found in: `/usr/local/etc/php/{{version}}`

Install Xdebug: `pecl install xdebug`, it will automaticlly install and configure in `php.ini`, you can check the `php.ini` to verify the installation: `zend_extension="xdebug.so"`.

Install Composer: `brew install composer`, configure the repo: `composer config -g repo.packagist composer https://packagist.phpcomposer.com`

## Ruby

Installation through: `brew install ruby`, you may need configure the `PATH` with: `echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc`.

Configure `gem` to use the remote repo:

```sh
# add TUNA mirror & remove default origin
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
# list the mirror config
gem sources -l

bundle config mirror.https://rubygems.org https://mirrors.tuna.tsinghua.edu.cn/rubygems
```

Configure the `GEM_HOME` variable:

```sh
export GEM_HOME=$HOME/.gem
export PATH=$GEM_HOME/bin:$PATH
```

Install [CocoaPods](https://cocoapods.org/): `gem install cocoapods --user-install`. Setup CocoaPods master repo: `pod setup`.

Configure the pods remote repo:

```sh
cd ~/.cocoapods/repos
pod repo remove master
git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git master
```

Then, add below lines as the first line in each project's `podFile`:

```ruby
source 'https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git'
```
