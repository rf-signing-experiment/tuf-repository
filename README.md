# TUF repository

A [TUF](https://theupdateframework.io) repository managed with
[tuf-ci](https://github.com/rf-signing-experiment/tuf-ci): metadata is signed on YubiKeys
by its signers, and administered through pull requests.

- `metadata/` — TUF metadata. Written by `tuf-sign` and by CI, never by hand.
- `targets/` — the artifacts this repository vouches for.

## Signing

Clone this repository and install `tuf-sign`. Then:

```console
$ tuf-sign key      # check your YubiKey is set up
$ tuf-sign          # list the signing events waiting on you, and act on one
```

A pull request labelled *Signing event* will say who has yet to sign. It cannot be merged
until the `tuf-ci/signatures` check passes.

## Publishing an artifact

An ordinary commit on a branch whose name starts with `sign/`:

```console
$ git switch -c sign/add-something origin/main
$ cp something targets/
$ git add targets && git commit -m 'Add something' && git push -u origin HEAD
```

CI turns that into a metadata change, opens a pull request, and asks the signers of the
owning role to sign it.

## Using this repository as a template

1. Create a repository from this template. Keep `.github/`, `.gitattributes` and
   `targets/`; delete `metadata/`, which belongs to *this* repository's signers.
2. Create a GitHub App in the owning organisation with **Contents**, **Pull requests** and
   **Checks** set to *Read and write*, install it on the new repository, and set the
   `TUF_CI_APP_CLIENT_ID` variable (the App's Client ID, `Iv23li…`) and the
   `TUF_CI_APP_PRIVATE_KEY` secret. Installing the App on the repository is a separate
   step from creating it, and skipping it makes the workflow fail with a 404.
3. Repoint the `uses:` in `.github/workflows/signing-event.yml` at the `tuf-ci` commit you
   want to run.
4. Require the `tuf-ci/signatures` check in branch protection on `main`.
5. Push all of the above to `main`, and only then run `tuf-sign init sign/init`.

Step 5 is in that order for a reason: a push event runs the workflow as it exists in the
pushed commit, so a `sign/*` branch cut from a `main` without the workflow will push
successfully and report nothing.
