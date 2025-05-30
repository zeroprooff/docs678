---
title: Celo Release Process for CeloCLI and ContractKit
description: Details of the release process for updating CeloCLI and ContractKit on the Celo platform.
---

# Release Process for CeloCLI and ContractKit

Details of the release process for updating CeloCLI and ContractKit on the Celo platform.

:::warning
As of block height 31,056,500 (March 26, 2025, 3:00 AM UTC), Celo is no longer a standalone Layer 1 blockchain—it is now an Ethereum Layer 2!
Some documentation may be outdated as updates are in progress. If you encounter issues, please [file a bug report](https://github.com/celo-org/docs/issues/new/choose).

For the most up-to-date information, refer to our [Celo L2 documentation](https://docs.celo.org/cel2).
:::

---

## Versioning

Use the standard MAJOR.MINOR.PATCH semantic versioning scheme described at [semver.org](http://semver.org).

New releases can be expected as follows:

- Major releases: approximately yearly
- Minor releases: approximately 8 times a year
- Patch releases: as needed

Development builds will be identified as such: `x.y.z-dev`, and will be published as `x.y.z` when stable.

## Identifying releases

### NPM

You can find the npm packages in the following places:

- [@celo/celocli](https://www.npmjs.com/package/@celo/celocli)
- [@celo/contractkit](https://www.npmjs.com/package/@celo/contractkit)

### Github tags

To identify the commits included in a specific release and see which new features were added or bugs fixed, please refer to the [release notes](https://github.com/celo-org/celo-monorepo/releases) in the monorepo. Also to keep track of continual updates to the stable and dev versions of the packages, each package has a `CHANGELOG.md` file: [Celocli](https://github.com/celo-org/developer-tooling/blob/master/packages/cli/CHANGELOG.md) and [Contractkit](https://github.com/celo-org/developer-tooling/blob/master/packages/sdk/contractkit/CHANGELOG.md).
All releases should be tagged with the version number, e.g. `contractkit-vX.Y.Z`. Each release should include a summary of the release contents, including links to pull requests and issues with detailed description of any notable changes.

### Communication

The community will be notified of package updates through the following channels:

For all releases:

- Each package’s `CHANGELOG.md` file, as mentioned above
- Github releases page, as mentioned above
- [Discord](https://chat.celo.org): #developer-chat, #mainnet, and #sdk

For major releases:

- Twitter: [@CeloDevs](https://x.com/CeloDevs)
- Mailing list: cLabs’ Tech Sync
- [Celo Forum](https://forum.celo.org/)


## Testing

All builds of these packages are automatically tested for performance and backwards compatibility in CI. Any regressions in these tests should be considered a blocker for a release.
Minor and major releases are expected to go through additional rounds of manual testing as needed to verify behavior under stress conditions.

:::warning

Work in progress

:::

## Promotion process

- For a patch release: The first step of this process should be a commit that changes the version number encoded in the source from `x.y.z-dev` to `x.y.z+1-dev` and the final step should change the published version number from `x.y.z-1` to `x.y.z`.
- For minor releases, the same process should be followed, except the `y` value would increment, and the `z` value would become 0.
- For major releases, the same process should be followed, except the `x` value would increment, and `y` and `z` values would become 0.

Only one commit should ever have a non-dev tag at any given version number. When that commit is created, a tag should be added along with release notes. Once the tag is published it should not be reused for any further release or changes.

### Emergency patches

Bugs which affect the security, stability, or core functionality of the network may need to be released outside the standard release cycle. In this case, an emergency patch release should be created on top of all supported minor releases which contains the minimal change and corresponding test for the fix. An emergency patch retro will also be published, and will include information such as why the patch was necessary and what code changes it includes.

## Vulnerability Disclosure

Vulnerabilities in any of these releases should be disclosed according to the [security policy](https://github.com/celo-org/celo-blockchain/blob/master/SECURITY.md).

## Dependencies

- @celo/mobile - Dappkit relies on this
- Celocli
- All the packages under the ["SDK" folder](https://github.com/celo-org/developer-tooling/tree/master/packages/sdk) -- These all rely on each other quite a bit, so triple-check that these packages weren’t affected by a change in another.

## Dependents

- Celocli
- All the packages under the ["SDK" folder](https://github.com/celo-org/developer-tooling/tree/master/packages/sdk)
