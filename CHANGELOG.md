# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.11]

### Fixed

- `docker_stack_deploy`, `docker_compose_deploy`: read the epoch from `ansible_facts['date_time']` instead of the injected `ansible_date_time` fact. ansible-core deprecates the `INJECT_FACTS_AS_VARS` default of `True` and will stop injecting facts as top-level `ansible_*` variables in 2.24, which would leave `ansible_date_time` undefined and break the roles. This also removes the deprecation warning
- `docker_stack_deploy`, `docker_compose_deploy`: only run `find` on the tmp dest folder when it is a directory, removing the "not a directory" warning when the folder does not exist yet

## [1.0.10]

### Added

- Add `docker_compose_deploy` role, adapting `docker_stack_deploy`'s deploy mechanism to plain `docker compose` (via `community.docker.docker_compose_v2`) instead of Docker Swarm

## [1.0.7]

### Added

- Add `detach` option to `docker_stack_deploy` role to control detached mode of docker stack deployment (default: true)

## [1.0.6]

### Fix

- Add `with-registry-auth` to docker deploy

## [1.0.5]

### Added

- Option for keeping stack files
- Add sanity checks

## [1.0.1]

### Added

- Two roles:
  - docker_stack_deploy
  - ssh_agent

### Changed

### Removed

