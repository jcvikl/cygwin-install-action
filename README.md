Install on Win https://www.cygwin.com/
============================
1. The ln command links the file designated in the SourceFile parameter to the file designated by the TargetFile parameter or to the same file name in another directory specified by the TargetDirectory parameter. By default, the ln command creates hard links.
2. "You would not be corrupting Cygwin if you copied files to /home. But you might prefer to let Cygwin put you in /home/{yourusername} and then create symlinks to your Windows' Documents, Pictures, etc.

ln -s /cygdrive/C/Users/yourusername/Documents
ln -s /cygdrive/C/Users/yourusername/Pictures
#...etc..."

3. How to Use Cygwin https://www.wikihow.com/Use-Cygwin

Explore the basic commands. Some of the basic commands are listed in the table below alongside their corresponding command in Windows.
| Action	| Windows	| Cygwin
| list directory	| dir	| ls
| clear console	| cls	| clear
| copy file(s)	| copy	| cp
| move file(s)	| move	| mv
| delete file(s)	| del	| rm
| create directory	| md	| mkdir
| remove directory	| rd	| rm -rf
| change current directory	| cd	| cd
| current directory	| cd,  chdir	| pwd
| search	| find	| grep
| concatenate	| cat	| cat
| permissions	| chmod	| chmod
| display/output text	| echo	e| cho





Cygwin Install GitHub Action
============================

This GitHub Action can be used in a workflow to install Cygwin.

e.g.

    - run: git config --global core.autocrlf input

    - uses: actions/checkout@v2

    - uses: cygwin/cygwin-install-action@master

    - run: bash tests/script.sh  # see footnote [1]

Please fix my terrible cargo-cult PowerShell.

Parameters
----------

| Input       | Default                                      | Description
| ----------- | -------------------------------------------- | -----------
| platform    | x86_64                                       | Install the x86 or x86\_64 version of Cygwin.
| packages    | *none*                                       | List of additional packages to install.
| install-dir | C:\cygwin                                    | Installation directory
| site        | http://mirrors.kernel.org/sourceware/cygwin/ | Mirror sites to install from, separated by whitespace
| pubkeys     | *none*                                       | Absolute paths of extra public key files (RFC4880 format), separated by whitespace
| check-sig   | true                                         | Whether to check the setup.ini signature
| add-to-path | true                                         | Whether to add Cygwin's `/bin` directory to the system `PATH`

Line endings
------------

If you're going to use `actions/checkout` in your workflow, you should
precede that with

    - run: git config --global core.autocrlf input

to ensure that any shell scripts etc. in your repository don't get checked out
with `\r\n` line endings (leading to `'\r': command not found` errors).

Likewise, if you have multiple lines of shell script in a YAML block for `run:`
in your workflow file, the file this is written into on the runner ends up with
`\r\n` line endings.

You can use `>-` (rather than `|`) to ensure that it doesn't contain any
newlines.

Alternatively, you can also use:

- `igncr` in the `SHELLOPTS` environment variable
- invoke `bash` with `-o igncr`

PATH
----

By default, this action prepends Cygwin's /usr/bin directory to the PATH.

However, if you want to ensure that PATH only contains Cygwin executables,
and other stuff installed in the VM image isn't going to get picked up:

- Set PATH to something like `/usr/bin:$(cygpath ${SYSTEMROOT})/system32` in
  your shell script

or,

- Put `CYGWIN_NOWINPATH=1` into the environment
- start a login shell with `bash --login`
- because the profile script from does `cd ${HOME}`, either:
  * `cd ${GITHUB_WORKSPACE}` in your shell script, or
  * prevent the profile script from changing directory by putting
    `CHERE_INVOKING` into the environment

If you want the opposite – the system PATH to remain unchanged by this action – add `add-to-path: false` to the action settings.

Symlinks
--------

Cygwin's `setup` creates Cygwin-style symlinks by default, and some
executables (e.g. `python`) are symlinks.

Since CMD and PowerShell don't understand those symlinks, you cannot run
those executables directly in a `run:` in your workflow. Execute them via
`bash` or `env` instead.

Alternatively, putting e.g. `CYGWIN=winsymlinks:native` into the workflow's
environment works, since setup now honours that.

Mirrors and signatures
----------------------

You probably don't need to change the setting for `site`, and you shouldn't
change `check-sig` unless you're very confident it's appropriate and necessary.
These options are very unlikely to be useful except in some very isolated
circumstances, such as using the [Cygwin Time
Machine](http://www.crouchingtigerhiddenfruitbat.org/Cygwin/timemachine.html).

[1] The
[Workflow documentation](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#exit-codes-and-error-action-preference)
suggests you should also use bash options `-eo pipefail`, omitted here for clarity
