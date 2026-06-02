# PR: Fix token refresh race and simplify session handling

## Summary

This PR rewrites the refresh-token rotation path to avoid a race where two refresh requests could both succeed. It also simplifies session middleware by moving token replacement into a single helper and removing some duplicated null checks.

## Changes

- replace `tokenStore.update()` with `replaceSessionTokens()`
- eagerly rotate refresh tokens after verification
- simplify middleware session lookup path
- update tests to reflect the new refresh flow

## Motivation

We saw intermittent reports that users could remain logged in on multiple clients after refresh, and a refresh request replay could occasionally succeed if requests landed close together.

## Risk

Main risk is auth/session behavior regressions. I updated the auth and middleware tests.
