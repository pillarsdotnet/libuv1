# libuv1 — backport packaging

Debian's **libuv1 1.52.1-4** rebuilt for Ubuntu series that ship something
older, and published to the Forgejo Debian registry.

Resolute (26.04) carries `libuv1-dev 1.51.0-2ubuntu1`. Node 24 wants 1.52, so
the [nodejs backport](../nodejs) needs this one first.

## What this repository contains

**Only `debian/`.** The upstream source is *not* committed — it is fetched at
build time from the `.dsc` named in `.forgejo/upstream.conf`:

```sh
UPSTREAM_DSC=https://deb.debian.org/debian/pool/main/libu/libuv1/libuv1_1.52.1-4.dsc
```

That keeps the repository at a couple of hundred kilobytes instead of carrying
a source tree, and makes rebasing onto a newer upstream a one-line change.
`dpkg-source -x` verifies the tarballs against the `.dsc`'s own SHA-256 sums on
every build, so nothing is trusted on the strength of a URL alone.

The packaging delta against Debian's `-4` is currently **the changelog only** —
this is a straight rebuild, not a patched fork. If that stops being true, the
difference will live in `debian/` and show up in `git diff`, which is the point
of having this repository at all.

## Versioning

`<upstream>~ubuntu<release>~ppaN`, for example
`1.52.1-4~ubuntu26.04~ppa1`. `N` is read back from what the registry already
publishes for that series, so rebuilding an unchanged upstream bumps the
revision rather than colliding. A `409` on upload means that exact version is
already there and is not treated as an error.

## Rebasing onto a newer upstream

1. Find the new `.dsc` on <https://deb.debian.org/debian/pool/main/libu/libuv1/>.
2. Update `UPSTREAM_DSC` in `.forgejo/upstream.conf`.
3. Push. CI rebuilds for every series in the matrix.

Nothing in `debian/` needs touching unless Debian's packaging changed in a way
this backport has to work around — and then that change belongs in `debian/`,
with a changelog entry saying why.

## Installing

The registry is an apt source inside the build containers; see
`kubuntu_bootstrap_ansible/roles/sources`. The packages are
`libuv1t64`, `libuv1-dev` and `libuv1-doc`.

## Branches

- `main` — the packaging. CI builds from here.
- `github` — `main` with `.forgejo/` pruned from every commit, pushed to the
  read-only GitHub mirror. Generated; never commit to it.
