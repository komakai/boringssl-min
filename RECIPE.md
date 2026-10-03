# RECIPE: boringssl-min

How the `main-min` branch of this repository was made from upstream, and
everything that differs from it. It is a submodule of webrtc-third_party-min (at `boringssl/src`) for [webrtc-min](https://github.com/komakai/webrtc-min).

## Upstream

- Repository: https://boringssl.googlesource.com/boringssl.git
- Revision: 5fbad2285b096858fc9afa3e4c949fde39452070 (2026-09-23T08:19:25-07:00)

The first commit on `main-min` is an unmodified snapshot of that revision, so
`git diff <first commit> main-min` shows every change made here.

## Stripped

Removed because nothing in the build needs them (run from the repository
root on the snapshot):

```sh
# Tests, test vectors, fuzzers, benchmarks, the bssl tool, developer scripts,
# Rust bindings, vendored test-only libraries, docs and other build systems.
git rm -rq agents bench docs fuzz infra rust tool util ssl/test pki/testdata \
  crypto/test gen/test_support third_party/benchmark third_party/googletest \
  third_party/wycheproof_testvectors cmake
git rm -q BUILD.bazel MODULE.bazel MODULE.bazel.lock CMakeLists.txt build.json \
  codereview.settings go.mod go.sum PRESUBMIT.py gen/sources.bzl gen/sources.gni \
  gen/sources.mk gen/sources.json API-CONVENTIONS.md BREAKING-CHANGES.md \
  BUILDING.md CONTRIBUTING.md FUZZING.md INCORPORATING.md PORTING.md \
  SANDBOXING.md STYLE.md
git ls-files | grep -E '(_test|_tests|_test_util)\.(cc|c|h)$|/test/|_tests?\.txt$|(^|/)test_?vectors?|(^|/)testdata/' \
  | xargs -r git rm -q
```

## Deviations from upstream

- **CMake** (commit "Add CMake build"): `CMakeLists.txt` replaces upstream's
  (which also builds tests, tools and the FIPS module variants) with one
  `boringssl` library following Chromium's `third_party/boringssl/BUILD.gn`:
  the `bcm`, `crypto`, `ssl` and `pki` sources plus the assembly for all
  platforms (each `.S` file assembles to nothing except on its own platform),
  with `BORINGSSL_IMPLEMENTATION` and `OPENSSL_SMALL`. The lists come from
  `gen/sources.cmake`, as gn's come from `gen/sources.gni`. `decrepit` isn't
  built (Chromium doesn't build it either), and Windows NASM assembly isn't
  supported.
- `gen/sources.cmake` still lists the removed test sources and test data;
  they aren't used.

## Updating to a new upstream revision

`main-min` isn't a git fork of upstream (no upstream history), so updates are
re-applied rather than merged:

1. Replace the tree with upstream at the new revision and commit it as
   "Snapshot of upstream at <rev>" (`git rm -rq . && git archive` of the new
   revision, or a copy of a fresh checkout).
2. Re-run the strip commands above and commit.
3. Re-apply the deviation commits (`git cherry-pick` them from the previous
   `main-min` history) and fix up any conflicts.
4. Update the source lists in the CMake files for files upstream added,
   removed or renamed (compare with the new BUILD.gn in the snapshot commit),
   then build webrtc-min and compare with a gn build.
