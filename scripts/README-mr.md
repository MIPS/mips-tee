Using myrepos to help manage multiple repositories
==================================================

This is a README regarding the myrepos `mr` tool and `do-mips-tee-mrconfig`
script.

To populate the mips-tee directory with the necessary external repositories run
the initialization script from the top-level directory "<mips_tee_directory>".

    cd <mips_tee_directory>
    ./scripts/do-mips-tee-mrconfig
    ./tools/bin/mr list
    ./tools/bin/mr status

The initialization script installs the myrepos tool `mr` from the
[myrepos git](https://myrepos.branchable.com) and uses the `mr bootstrap` and
`mr checkout` commands to clone the external repositories and checkout the
required branches.  The checkout command is only called when the external
repository does not exist.

When done run `mr list` and `mr status` to verify that all the repositories
have been properly populated. If the network connection was lost during
initialization it will identify any partially populated external repositories
by listing missing files in those repositories (see "Failed checkout" below).

It is prudent to run `mr help` and read the section "UNTRUSTED MRCONFIG FILES"
and check the contents of "$HOME/.mrtrust" to ensure they are correct. The
top-level "<mips_tee_directory>/.mrconfig" file can be modified to customize
how repositories are acted on.

Afterwards the `mr` tool can be used to operate on one or all of the top-level or
external repositories:

    mr help
    mr [-d dir] status
    mr [-d dir] update


Common issues
=============

- Failed checkout, delete directory to force new checkout:

The checkout command is only called when the repository doesn't yet exist. If
the `do-mips-tee-mrconfig` script fails after it has already created a
directory for an external repository but before finishing the checkout, the
directory may be deleted manually to force the next invocation to repeat the
failed checkout operation on the external repository.

- Identifying generated files for manual cleanup:

Run `mr list` for a list of paths `mr` acts on and has created.
Run `do-mips-tee-mrconfig clean` to remove `mr` and some generated files.
Check the contents of "$HOME/.mrtrust" and delete any out of date entries.

- Illegal command, untrusted config file:

After the first run "scripts/.mips_tee_mrconfig" is copied to
"<mips_tee_directory>/.mrconfig".  Subsequent invocations will complain that
.mrconfig is untrusted and display the following message.

>   mr: illegal checkout command in untrusted mips-tee/.mrconfig
>   (To trust this file, list it in ~/.mrtrust.)

Add .mrconfig to .mrtrust in order to continue using `mr`, or specify
`mr --trust-all`.

- mr command not found, error message:
>   sh: 51: mr: command not found

The `mr` tool calls itself recursively but `mr` is not in the shell PATH at the
time. This problem occurs with some shells when the newly installed `mr` tool can
not be found. Add $TOOLS_BIN/mr to your PATH to resolve this.
