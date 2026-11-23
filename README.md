# libuv1 — `backport` packaging

Debian's **libuv1 1.53.0-1** rebuilt for Ubuntu series that ship something
older, and published to the `Forgejo` Debian registry.

`Resolute` (26.04) carries `libuv1-dev 1.51.0-2ubuntu1`. Node 24 wants 1.52, so
the [`nodejs`](../nodejs) build needs this one first.

## What this repository contains

**Only `debian/`.** The upstream source is _not_ committed — it is fetched at
build time from the `.dsc` named in `.forgejo/upstream.conf`:

```sh
UPSTREAM_DSC=https://deb.debian.org/debian/pool/main/libu/libuv1/libuv1_1.53.0-1.dsc
```

That keeps the repository at a couple of hundred kilobytes instead of carrying
a source tree, and makes rebasing onto a newer upstream a one-line change.
`dpkg-source -x` verifies the tarballs against the `.dsc`'s own SHA-256 sums on
every build, so nothing is trusted on the strength of a URL alone.

The packaging delta against Debian is currently **nothing at all** — `debian/`
is Debian's `1.53.0-1` verbatim, and CI adds the `backport` changelog entry at
build time. This is a straight rebuild, not a patched fork. If that stops being
true, the difference will live in `debian/` and show up in `git diff`, which is
the point of having this repository at all.

**Debian removes superseded versions from its pool.** The first build here
404'd because it still pointed at `1.52.1-4`, which had been dropped. If a
build fails to fetch, look at the pool directory above for the current `.dsc`
before assuming anything else is wrong.

## The build drops privileges

`dpkg-buildpackage` runs as an unprivileged `builder` user, not as root.
`libuv`'s test suite refuses to run as root and says so, which failed the first
build here:

```text
The libuv test suite cannot be run as root.
0% tests passed, 2 tests failed out of 2
```

Launchpad never hit this because its builders are unprivileged; a container is
root by default, so the build has to drop privileges itself.
`DEB_BUILD_OPTIONS=nocheck` would also make the build pass — by not running the
thing that was complaining — which is why it is not what this does.

## Versioning

`<upstream>~ubuntu<release>~ppaN`, for example
`1.52.1-4~ubuntu26.04~ppa1`. `N` is read back from what the registry already
publishes for that series, so rebuilding an unchanged upstream bumps the
revision rather than colliding. A `409` on upload means that exact version is
already there and is not treated as an error.

## Rebasing onto a newer upstream

1. Find the new `.dsc` on <https://deb.debian.org/debian/pool/main/libu/libuv1/>.
2. Update `UPSTREAM_DSC` in `.forgejo/upstream.conf`.
3. Replace `debian/` with the new upstream's, since this `backport` carries no
   patches of its own: `dget -du <dsc> && dpkg-source -x *.dsc src` then copy
   `src/debian` over. Debian's packaging moved 17 files between `1.52.1-4` and
   `1.53.0-1`, so carrying the old one forward would not have been a rebase.
4. Push. CI rebuilds for every series in the matrix.

## Installing

The registry is an apt source inside the build containers; see
`kubuntu_bootstrap_ansible/roles/sources`. The packages are
`libuv1t64`, `libuv1-dev` and `libuv1-doc`.

## Branches

- `main` — the packaging. CI builds from here.
- `github` — `main` with `.forgejo/` pruned from every commit, pushed to the
  read-only GitHub mirror. Generated; never commit to it.
