# yarn-workspace-portal-issue
Reproduction of an issue I ran into with portalling a package that exists inside of a workspace (and depends on other workspace dependencies) that broke in yarn v3.2.0.

* All packages us `nodeLinker: node-modules` - I haven't tested if other `nodeLinker` values suffer from issues on this reproduction.
* In this reproduction, the packages `3.1.1` and `3.2.0` have the same structure, their only difference is the version of Yarn used.

## Structure
* First, I have a `workspace` package, with two packages inside it in a workspace, `workspace-a` and `workspace-b`.
* `workspace-b` has a dependency on `workspace-a`, defined as `"workspace:*"`
* Then, I have a package outside of the `workspace` package that is using `workspace-b`, with a `portal:` leading to `../workspace/workspace-b`.
* Since `workspace-b` has a dependency on `workspace-a`, we also need to add a `resolution` key to the package.json that defines `workspace-b/workspace-a` as using a `portal:` leading to `../workspace/workspace-a`.

## Steps
1. Run `yarn install` inside the `3.1.1` package, and observe the output throws no errors.
2. Run `yarn install` inside the `3.2.0` or `canary` package, and observe the output is now throwing an unexpected error:
```
➤ YN0001: │ Error: Assertion failed: Writing attempt prevented to yarn-workspace-portal-issue/workspace/workspace-b/node_modules/workspace-a which is outside project root: yarn-workspace-portal-issue/3.2.0
    at fb (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:712:1725)
    at nce (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:712:2298)
    at T_e (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:712:6802)
    at rce.finalizeInstall (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:697:26714)
    at async ze.linkEverything (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:441:14959)
    at async yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:444:4324
    at async Je.startSectionPromise (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:413:3303)
    at async ze.install (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:444:4100)
    at async yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:501:12366
    at async Function.start (yarn-workspace-portal-issue\3.2.0\.yarn\releases\yarn-3.2.0.cjs:413:2373)
```

## Other things I tried but didn't fix the issue
* Adding a dependency on `workspace-a` that has the same `portal:`
* Adding a `packageExtensions` key to the `.yarnrc.yml`
* Updated to canary (it appears to just be 3.2.0 at the moment anyway?)
