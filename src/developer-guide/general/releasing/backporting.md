# Backporting simple fixes

## Overview
| Method | Complexity | Time | Notes |
| --- | --- | --- | --- |
| [COPR](https://copr.fedorainfracloud.org/coprs/g/osbuild/) | Easy | Immediate | Unofficial, but the fix is already there |
| Unsigned RPM | Easy | 1 week | Unofficial in that it's unsigned |
| Async Update to z-stream | Difficult | 2 weeks | Requires sign-off from many and could be rejected |
| Batch Update to z-stream | Medium | up to 8 weeks | Red Hat's preferred method |

## How to

### COPR

1. Share the correct COPR package version and URL

### Unsigned RPM

1. Take the latest spec file from downstream
2. Create a downstream patch and add it
3. Create a scratch build in Brew
