# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.4] - 2026-03-02

### Fixed

- Support wildcard (`*/*`) and non-JSON content types in response parsing
  - Fixes missing Success Response Schema for APIs using `*/*` content type
- Use relative paths for cross-directory schema links
  - Schema documents in different subdirectories now correctly link to each other

## [0.2.3] - 2025-XX-XX

### Fixed

- Include apiKey header name in authentication output

## [0.2.2] - 2025-XX-XX

### Changed

- Build tooling improvements

## [0.2.0] - 2025-XX-XX

### Added

- Initial release with OpenAPI to Agent Skills conversion
- Support for OpenAPI 3.0 specifications
- Progressive disclosure documentation structure
- Schema grouping by prefix
- Multi-language support (English, Chinese, Japanese)
