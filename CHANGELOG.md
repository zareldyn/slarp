# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [Unreleased]

### Added

- Static folder Apache can serve directly (bind mount from host)
- Auto mount of web_static_* volumes at startup


## [1.3.0] - 2018-06-26

### Added

- proxy_wstunnel enabled in Apache, so WebSocket has a chance to work

### Fixed

- Error in documentation

### Changed

- is-running is safer to use
- The stop command always tries to remove the container, whatever is-running says


## [1.2.0] - 2018-04-08

### Added

- Custom persistent data location


## [1.1.1] - 2018-03-29

### Added

- This changelog

### Fixed

- is-running now correctly detects SLARP being executed


## [1.1.0] - 2018-03-24

### Added

- is-running utility

### Changed

- Scripts for the management of the service are safer to use
- and they all use the same simple policy for message output


## [1.0.0] - 2018-03-18

### Added

- Dockerfile to make the SLARP image
- Basic scripts to manage the SLARP service
- Predefined directories for persistent data
- README (explaining is the most difficult part of this work!)
