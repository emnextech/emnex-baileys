# Changelog

All notable changes to **emnex-baileys** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.5.9] - 2026-05-29

### Changed
- **Rebrand to `emnex-baileys`** (maintained by Emnex Tech), forked from `gifted-baileys`.
- Updated package metadata: repository, homepage, author, and the published `files` list.
- Refreshed and expanded the README with a Features list, a complete copy-paste "Getting Started" bot, and improved npm-discoverable keywords.
- Updated the default newsletter/channel URL.

### Added
- This changelog.

## [2.5.x] - earlier

### Added
- **Status / Stories**: send to the status broadcast with `statusJidList`, plus improved
  multi-device status handling and the ability to post status updates directly to groups.
- **ID ↔ phone-number mapping**: resolve participant phone numbers in group messages and
  map user IDs to phone numbers for consistent sender information.

### Fixed
- Correctly identify messages sent by the bot in group chats.
- Make incoming message recipient IDs consistent with outgoing ones.
- Fix message replies by preventing unintended chat-ID conversion.
- Improve direct-message handling and `senderPn` accuracy for incoming DMs.

---

> **Note on versioning:** this project is a fork and inherits its version line from upstream.
> Releases prior to the rebrand were published under the `gifted-baileys` name. For the
> original Baileys history, see [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys).

[2.5.9]: https://github.com/emnextech/emnex-baileys/releases/tag/v2.5.9
