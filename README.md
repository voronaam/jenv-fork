# About this fork

This is a fork of [jenv project](https://github.com/jenv/jenv) to which I have applied many of the Pull Requests submitted to the upstream. It is intended as a personal fork for convenience, but feel free to use it, if you like it. You may also submit additional Pull Requests for the merge here.

# Master your Java Environment with jenv

Website : http://www.jenv.be

Maintainers : 
- [Gildas Cuisinier](https://github.com/gcuisinier/) (the upstream maintainer)
- [Lex Vorona](https://github.com/voronaam/) (this fork maintainer)

Future maintainer in discussion:
- [Benjamin Berman](https://github.com/doctorpangloss) (the other fork maintainer, to which I did not find a link sadly)

## What's jEnv ?

This is an updated fork of `jenv`, a beloved Java environment manager adapted from `rbenv`.

`jenv` gives you a few critical affordances for using `java` on development machines:

 - It lets you switch between `java` versions. This is useful when developing Android applications, which generally require Java 8 for its tools, versus server applications, which use later versions like Java 11.
 - It sets `JAVA_HOME` inside your shell, in a way that can be set globally, local to the current working directory or per shell.

However, this project does **not**:

 - Install `java` for you. Use your platform appropriate package manager to install `java`. On macOS, `brew` is recommended.

This document will show you how to install `jenv`, review its most common commands, show example workflows and identify known issues.

### Contents

 1. [Getting Started](#1-getting-started)
 2. [Example Workflows](#2-common-workflows)
 3. [Known Issues](#3-known-issues)

### 1. Getting Started

Follow the steps below to get a working `jenv` installation with knowledge of your `java` environment. Read all the code you execute carefully: a `$` symbol at the beginning of a line should be omitted, since it's meant to show you entering a command into your terminal and observing the response after the command.

#### 1.1 Installing `jenv`

On OSX, the simpler way to install jEnv is using [Homebrew](https://brew.sh) (This will install the vanilla upstream)

```bash
brew install jenv
```

Alternatively, and on Linux, you can install it from source :

```bash
git clone https://github.com/voronaam/jenv-fork.git ~/.jenv
# Shell: bash
echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(jenv init -)"' >> ~/.bash_profile
# Shell: zsh
echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(jenv init -)"' >> ~/.zshrc
```

Restart your shell by closing and reopening your terminal window or running `exec $SHELL -l` in the current session for the changes to take effect.

To verify `jenv` was installed, run `jenv doctor`. On a macOS machine, you'll observe the following output:

```bash
$ jenv doctor
[OK]	No JAVA_HOME set
[ERROR]	Java binary in path is not in the jenv shims.
[ERROR]	Please check your path, or try using /path/to/java/home is not a valid path to java installation.
	PATH : /Users/user/.jenv/libexec:/Users/user/.jenv/shims:/Users/user/.jenv/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
[OK]	Jenv is correctly loaded
```

Observe that `jenv` is correctly loaded but Java is not yet installed.

To make sure `JAVA_HOME` is set, make sure to enable the `export` plugin:

```bash
jenv enable-plugin export
exec $SHELL -l
```

Problem? Please visit the [Trouble Shooting](https://github.com/jenv/jenv/wiki/Trouble-Shooting) Wiki page.

Continue to the next section to install java.



**Untested**: While this fork has improved `fish` shell support, it has not been tested by this maintainer. To install `jenv` for Fish according to the contributor's instructions:

```sh
echo 'set PATH $HOME/.jenv/bin $PATH' >> ~/.config/fish/config.fish
echo 'status --is-interactive; and source (jenv init -|psub)' >> ~/.config/fish/config.fish
cp ~/.jenv/fish/jenv.fish ~/.config/fish/functions/jenv.fish
cp ~/.jenv/fish/export.fish ~/.config/fish/functions/export.fish
```

#### 1.2 Adding Your Java Environment

Use `jenv add` to inform `jenv` where your Java environment is located. `jenv` does not, by itself, install Java.

For example, on macOS, use `brew` to install the latest Java (OpenJDK 11) followed by the appropriate `jenv add PATH_TO_JVM_HOME` command to recognize it.

```bash
brew cask install java
jenv add $(/usr/libexec/java_home)
```

With macOS OpenJDK 11.0.2 installed, for example, either of these commands will add `/Library/Java/JavaVirtualMachines/openjdk-11.0.2.jdk/Contents/Home` as a valid JVM. Your JVM directory may vary!

Observe now that this version of Java is added to your `java versions` command:

```bash
$ jenv versions
* system (set by /Users/user/.jenv/version)
  11.0
  11.0.2
  openjdk64-11.0.2
```

By default, the latest version of Java is your `system` Java on macOS.

We'll now set a `jenv local VERSION` local Java version for the current working directory. This will create a `.java-version` file we can check into Git for our projects, and `jenv` will load it correctly **when a shell is started from this directory**.

```bash
$ jenv local 11.0.2
$ exec $SHELL -l
$ cat .java-version
11.0.2
```

Is `JAVA_HOME` set?

```bash
$ echo ${JAVA_HOME}
/Users/bberman/.jenv/versions/11.0.2
```

Yes! Observe that `JAVA_HOME` is set to a valid shim directory. Unlike the main repository's documentation we helpfully installed the `export` plugin, and we now have the most important `jenv` features covered.

If you executed this commands inside your `$HOME` directory, you can now delete `.java-version`:

```bash
rm .java-version
```

#### 1.3 Setting a Global Java Version

Use `jenv global VERSION` to set a global Java version:

```bash
jenv global 11.0.2
```

When you next open a shell or terminal window, this version of Java will be the default.

On macOS, this sets `JAVA_HOME` for GUI applications on macOS using `jenv macos-javahome`. Integrates [this tutorial](https://www.ibm.com/support/knowledgecenter/en/SSPJLC_7.6.2/com.ibm.si.mpl.doc/tshoot/ts_java_home.html) to create a file that does **not update dynamically** depending on what local or shell version of Java is set, only global.


#### 1.4 Setting a Shell Java Version

Use `jenv shell VERSION` to set the Java used in this particular shell session:

```bash
jenv shell 11.0.2
```

### 2 Common Workflows

These common workflows demonstrate how to use `jenv` to solve common problems.

#### 2.1 Using Two JVMs on macOS

Our goal is to have both the latest version of Java and JDK 8 installed at the same time. This is helpful for developing Android applications, whose build tools are sensitive to using an exact Java version.

We'll resume where we left off with Java 11.0.2 installed. Let's [install Java 8](https://stackoverflow.com/questions/24342886/how-to-install-java-8-on-mac) now:

```bash
brew cask install adoptopenjdk8
brew cask install caskroom/versions/adoptopenjdk8
```

This will install the latest version of Java 8 to a special directory in macOS. Let's see which directory that is:

```bash
$ ls -1 /Library/Java/JavaVirtualMachines 
adoptopenjdk-8.jdk
openjdk-11.0.2.jdk
```

Observe the `adoptopenjdk-8.jdk` directory. **Your exact version may vary**. We cannot retrieve this using `/usr/libexec/java_home`, unfortunately. We'll add the Java home directory using `jenv` so that it shows up in our `jenv versions` command:

```bash
$ jenv add /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/
openjdk64-1.8.0.222 added
1.8.0.222 added
1.8 added
$ jenv versions
* system
  1.8
  1.8.0.222
  openjdk64-1.8.0.222
  11.0
  11.0.2
  openjdk64-11.0.2
  oracle64-1.8.0.202-ea
```

#### 2.3 jenv update

Update jenv and all installed external plugins

    $ jenv update
    [OK]	Updating /home/bshell/.jenv...
    From https://github.com/gcuisinier/jenv
    * branch            master     -> FETCH_HEAD
    Already up-to-date.

#### 2.4 Other Workflows

Please contribute your own using a pull request!

### 3 Known Issues

Users seem to have issues using `jenv` with Fish. Please report any here.



### 4 This fork differences

- ~~Fix to the deprecated `jenv add <alias> <path>` functionality. Tested on Ubuntu/bash, MacOS/bash.~~ (applied upstream)
- Merged PR for variable safety in fish. [Original PR link](https://github.com/jenv/jenv/pull/52)
- Merged PR for the `jenv update` command. [Original PR link](https://github.com/jenv/jenv/pull/65). Tested on Ubuntu/bash.
- Merged PR for system's javahome. [Original PR link](https://github.com/jenv/jenv/pull/134). Tested on Ubuntu/bash.
- Merged PR for the `jenv remove-all` command. [Original PR link](https://github.com/jenv/jenv/pull/146). Merged with changes. Tested on Ubuntu/bash.
- Added `jenv remove-broken` command that removes all the versions, which were deleted from the OS and their symlinks are now broken.
- Merged PR for the `jenv remove` multiple arguments command. [Original PR link](https://github.com/jenv/jenv/pull/269/). Merged with changes. Tested on Ubuntu/bash.
- Added ability to specify version on the exec command as in `jenv exec --version 14 java -jar myjar.jar`. Tested on Ubuntu/bash.

#### 4.1 Other patchsets for future merges:

- [Maven versioning](https://github.com/jenv/jenv/pull/73)
- [Bootstrap command](https://github.com/jenv/jenv/pull/82)
