**More Robust Solution**

For _pip3_, use this:

```
pip3 freeze --local |sed -rn 's/^([^=# \t\\][^ \t=]*)=.*/echo; echo Processing \1 ...; pip3 install -U \1/p' |sh
```

For _pip_, just remove the 3s as such:

```
pip freeze --local |sed -rn 's/^([^=# \t\\][^ \t=]*)=.*/echo; echo Processing \1 ...; pip install -U \1/p' |sh
```

**OS X Oddity**

OS X, as of July 2017, ships with a very old version of [sed](https://en.wikipedia.org/wiki/Sed) (a dozen years old). To get extended regular expressions, use `-E` instead of `-r` in the solution above.

**Solving Issues with Popular Solutions**

This solution is well designed and tested1, whereas there are problems with even the most popular solutions.

-   Portability issues due to changing pip command line features
-   Crashing of [xargs](https://en.wikipedia.org/wiki/Xargs) because of common pip or pip3 child process failures
-   Crowded logging from the raw _xargs_ output
-   Relying on a Python-to-OS bridge while potentially upgrading it3

The above command uses the simplest and most portable _pip_ syntax in combination with _sed_ and _sh_ to overcome these issues completely. Details of the _sed_ operation can be scrutinized with the commented version2.

___

**Details**

\[1\] Tested and regularly used in a Linux 4.8.16-200.fc24.x86\_64 cluster and tested on five other Linux/Unix flavors. It also runs on Cygwin64 installed on Windows 10. Testing on iOS is needed.

\[2\] To see the anatomy of the command more clearly, this is the exact equivalent of the above pip3 command with comments:

```
# Match lines from pip's local package list output
# that meet the following three criteria and pass the
# package name to the replacement string in group 1.
# (a) Do not start with invalid characters
# (b) Follow the rule of no white space in the package names
# (c) Immediately follow the package name with an equal sign
sed="s/^([^=# \t\\][^ \t=]*)=.*"

# Separate the output of package upgrades with a blank line
sed="$sed/echo"

# Indicate what package is being processed
sed="$sed; echo Processing \1 ..."

# Perform the upgrade using just the valid package name
sed="$sed; pip3 install -U \1"

# Output the commands
sed="$sed/p"

# Stream edit the list as above
# and pass the commands to a shell
pip3 freeze --local | sed -rn "$sed" | sh
```

\[3\] Upgrading a Python or PIP component that is also used in the upgrading of a Python or PIP component can be a potential cause of a deadlock or package database corruption.