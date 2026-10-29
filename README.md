# chi NoCache request preconditions: validation

Candidate `110c345fa782031fe6d0836527c069dceea78243`; current upstream baseline `9f8adb1aabaa8a91180a821111d3f8e668d1c0bf`.

Based on [go-chi/chi#1098](https://github.com/go-chi/chi/pull/1098), commit `c5dfc5ec360f0b9532e21734388b96413eaca116`. The original Git author Deepak kudi, date and source trailer are retained in cherry-pick `058f25d9ad78cd6b52cee24156c379f44105e856`. The original PR account is `puneetdixit200`; these are attribution facts, not claims of endorsement or additional permission. MIT LICENSE is unchanged.

## Problem and increment

NoCache currently deletes conditional request headers before downstream handlers can evaluate them. This can bypass lost-update protection on PUT and change conditional range behavior. The original removal of request mutation is retained; existing response cache directives are unchanged. The increment is one explanatory godoc line and 11 real loopback HTTP scenarios through a chi router: eight standard-library ServeContent cases and three stateful PUT cases. The original direct header regression remains.

[RFC 9110, section 13.1](https://www.rfc-editor.org/rfc/rfc9110.html#section-13.1) defines these origin-server preconditions. NoCache configures response caching rather than implementing application-specific precondition evaluation.

## Completed validation

- Linux Go 1.24.13/1.25.14/1.26.10/1.27.3: both exact revisions ran the unchanged project `go get -d -t ./...; make test`, including race tests for both packages; candidate vet and build passed.
- Native Windows Go 1.26.10/1.27.3: both revisions passed full ordinary tests; candidate vet/build passed. Native Windows race/make and older Windows Go versions were **not** run on the local host.
- Before PR creation, the unchanged [own-fork workflow run 40974581830](https://github.com/jipeng6036-del/chi/actions/runs/40974581830) passed **all eight jobs**: Go 1.24.x/1.25.x/1.26.x/1.27.x × Ubuntu/Windows, including actual Windows make/race. Every job has 142 top-level PASS events and 262 subtest PASS events, no skips or failures. These are event counts including parent tests/examples, not 404 independent tests.
- Identical HTTP fixtures on the baseline produce seven failing scenarios and four passing controls; all 11 pass on the candidate. The same result was reproduced on Windows and Linux in a separate consumer module after installing actual local module ZIPs, with separate caches and no replace directive. The consumer uses `go test` to start actual TCP loopback HTTP servers; it is not a separate `go run` application or an external production acceptance test.
- The actual CONTRIBUTING command `goimports -w .` (pinned v0.38.0 before validation) passed on isolated exact baseline/candidate qualification copies, with zero changed bytes or modes. Changed-file diff output is empty. A diagnostic `goimports -h` version probe returned usage after the required tests; it is preserved and excluded from passing evidence. The same tool version was then verified using build info and the actual write entry succeeded.
- Baseline 110 files and candidate 111 files are bound to raw Git blobs and actual Linux modes. Installed module files are independently hashed. `VALIDATION_SUMMARY.json`, source manifests, `MODULE_PROOF.json` and command receipts describe the exact coverage.

## Reproduce project checks

Checkout either exact revision with the corresponding Go version installed:

```sh
go version
go get -d -t ./...
make test
go vet ./...
go build ./...
goimports -w .
```

`make test` cleans the test result cache, then runs `go test -race -v .` and `go test -race -v ./middleware`. Windows needs the normal make/C compiler environment used by the unchanged CI. For the negative control, copy only the candidate `middleware/nocache_test.go` into a separate baseline checkout and run `go test -count=1 -v ./middleware -run TestNoCache`; failures are expected, without changing production code.

## Reproduce installed-module HTTP checks

The `module-proxy` directory is a file-based Go module proxy for two unpublished local fixtures, `v5.0.0-baseline` and `v5.0.0-candidate`. The `consumer-*` directories contain the exact external-package tests that were run; they import the installed chi module rather than a replace directive. Run with Go 1.27.3 and separate caches. No model, paid provider, or external HTTP service is used.

POSIX from this proof directory:

```sh
proof="$(pwd)"
for stage in baseline candidate; do
  (cd "consumer-$stage" && \
   GOTOOLCHAIN=local GOSUMDB=off \
   GOMODCACHE="$proof/repro-cache-$stage" \
   GOPROXY="file://$proof/module-proxy" go test -count=1 -v ./...)
done
```

PowerShell from this proof directory:

```powershell
$proof = (Get-Location).Path
foreach ($stage in @('baseline', 'candidate')) {
  $env:GOTOOLCHAIN = 'local'
  $env:GOSUMDB = 'off'
  $env:GOMODCACHE = Join-Path $proof "repro-cache-$stage"
  $env:GOPROXY = ([Uri](Join-Path $proof 'module-proxy')).AbsoluteUri
  Push-Location "consumer-$stage"
  try { go test -count=1 -v ./... } finally { Pop-Location }
}
```

The baseline exits 1, the candidate 0. `GOSUMDB=off` applies only to these local unpublished fixtures; use a disposable shell and an already installed exact Go binary. The two fixture versions were not published to a registry.

## Provenance and limits

Original evidence is unchanged locally. Public text copies may normalize UTF-8/newlines/JSON formatting and replace only known host path prefixes. `PROVENANCE.json` binds original and public SHA-256 values; ZIPs and consumer source bytes are retained. `COMMANDS.json` contains selected actual validation receipts, with `$PROOF`, `$CHECKOUT`, `$GO126` and `$GO127` as private-path placeholders, not literal runnable command strings. Fork CI and later upstream PR CI/maintainer review are separate. There is no claim of personal CLA signing, human review, production concurrency acceptance or original-author approval. Codex assisted the additional regressions and validation.
