# Contributing to this manual {#chap-contributing}

The sources of the NixOS manual are in the [nixos/doc/manual](https://github.com/NixOS/nixpkgs/tree/master/nixos/doc/manual) subdirectory of the [Nixpkgs](https://github.com/NixOS/nixpkgs) repository.
This manual uses the [Nixpkgs manual syntax](https://nixos.org/manual/nixpkgs/unstable/#sec-contributing-markup).

You can quickly check your edits with the following:

```ShellSession
$ cd /path/to/nixpkgs
$ $EDITOR doc/nixos/manual/... # edit the manual
$ nix-build nixos/release.nix -A manual.x86_64-linux
```

If the build succeeds, the manual will be in `./result/share/doc/nixos/index.html`.

There's also [a convenient development daemon](https://nixos.org/manual/nixpkgs/unstable/#sec-contributing-devmode).

The above instructions don't deal with the appendix of available `configuration.nix` options, and the manual pages related to NixOS. These are built, and written in a different location and in a different format, as explained in the next sections.

## Development environment {#sec-contributing-development-env}

In order to reduce repetition, consider using tools from the provided development environment:

Load it from the NixOS documentation directory with

```ShellSession
$ cd /path/to/nixpkgs/nixos/doc/manual
$ nix-shell
```

To load the development utilities automatically when entering that directory, [set up `nix-direnv`](https://nix.dev/guides/recipes/direnv).

Make sure that your local files aren't added to Git history by adding the following lines to `.git/info/exclude` at the root of the Nixpkgs repository:

```
/**/.envrc
/**/.direnv
```

### `devmode` {#sec-contributing-devmode}

Use [`devmode`](https://github.com/NixOS/nixpkgs/blob/master/pkgs/by-name/de/devmode/README.md) for a live preview when editing the manual.

## Testing redirects {#sec-contributing-redirects}

Once you have a successful build, you can open the relevant HTML (path mentioned above) in a browser along with the anchor, and observe the redirection.

Note that if you already loaded the page and *then* input the anchor, you will need to perform a reload. This is because browsers do not re-run client JS code when only the anchor has changed.

## Contributing to the `configuration.nix` options documentation {#sec-contributing-options}

The documentation for all the different `configuration.nix` options is automatically generated by reading the `description`s of all the NixOS options defined at `nixos/modules/`. If you want to improve such `description`, find it in the `nixos/modules/` directory, and edit it and open a pull request.

To see how your changes render on the web, run again:

```ShellSession
$ nix-build nixos/release.nix -A manual.x86_64-linux
```

And you'll see the changes to the appendix in the path `result/share/doc/nixos/options.html`.

You can also build only the `configuration.nix(5)` manual page, via:

```ShellSession
$ cd /path/to/nixpkgs
$ nix-build nixos/release.nix -A nixos-configuration-reference-manpage.x86_64-linux
```

And observe the result via:

```ShellSession
$ man --local-file result/share/man/man5/configuration.nix.5
```

If you're on a different architecture that's supported by NixOS (check file `nixos/release.nix` on Nixpkgs' repository) then replace `x86_64-linux` with the architecture. `nix-build` will complain otherwise, but should also tell you which architecture you have + the supported ones.

## Contributing to `nixos-*` tools' manpages {#sec-contributing-nixos-tools}

The manual pages for the tools available in the installation image can be found in Nixpkgs by running (e.g for `nixos-rebuild`):

```ShellSession
$ git ls | grep nixos-rebuild.8
```

Man pages are written in [`mdoc(7)` format](https://mandoc.bsd.lv/man/mdoc.7.html) and should be portable between mandoc and groff for rendering (except for minor differences, notably different spacing rules.)

For a preview, run `man --local-file path/to/file.8`.

Being written in `mdoc`, these manpages use semantic markup. This following subsections provides a guideline on where to apply which semantic elements.

### Command lines and arguments {#ssec-contributing-nixos-tools-cli-and-args}

In any manpage, commands, flags and arguments to the *current* executable should be marked according to their semantics. Commands, flags and arguments passed to *other* executables should not be marked like this and should instead be considered as code examples and marked with `Ql`.

- Use `Fl` to mark flag arguments, `Ar` for their arguments.
- Repeating arguments should be marked by adding an ellipsis (spelled with periods, `...`).
- Use `Cm` to mark literal string arguments, e.g. the `boot` command argument passed to `nixos-rebuild`.
- Optional flags or arguments should be marked with `Op`. This includes optional repeating arguments.
- Required flags or arguments should not be marked.
- Mutually exclusive groups of arguments should be enclosed in curly brackets, preferably created with `Bro`/`Brc` blocks.

When an argument is used in an example it should be marked up with `Ar` again to differentiate it from a constant. For example, a command with a `--host name` option that calls ssh to retrieve the host's local time would signify this thusly:
```
This will run
.Ic ssh Ar name Ic time
to retrieve the remote time.
```

### Paths, NixOS options, environment variables {#ssec-contributing-nixos-tools-options-and-environment}

Constant paths should be marked with `Pa`, NixOS options with `Va`, and environment variables with `Ev`.

Generated paths, e.g. `result/bin/run-hostname-vm` (where `hostname` is a variable or arguments) should be marked as `Ql` inline literals with their variable components marked appropriately.

 - When `hostname` refers to an argument, it becomes `.Ql result/bin/run- Ns Ar hostname Ns -vm`
 - When `hostname` refers to a variable, it becomes `.Ql result/bin/run- Ns Va hostname Ns -vm`

### Code examples and other commands {#ssec-contributing-nixos-tools-code-examples}

In free text names and complete invocations of other commands (e.g. `ssh` or `tar -xvf src.tar`) should be marked with `Ic`, fragments of command lines should be marked with `Ql`.

Larger code blocks or those that cannot be shown inline should use indented literal display block markup for their contents, i.e.

```
.Bd -literal -offset indent
...
.Ed
```

Contents of code blocks may be marked up further, e.g. if they refer to arguments that will be substituted into them:

```
.Bd -literal -offset indent
{
  config.networking.hostname = "\c
.Ar hostname Ns \c
";
}
.Ed
```
