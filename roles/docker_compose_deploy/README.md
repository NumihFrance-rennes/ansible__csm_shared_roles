# Ansible role : docker_compose_deploy

This role allows to prepare a folder containing a docker compose project and call `docker compose up` on it.

## Requirements

- The `community.docker` collection must be installed on the controller (provides the `community.docker.docker_compose_v2` module used by this role).

## ⚙️ Variables

Variables :

| Variable name             | Description                                                                 | Type    | Default value                                         |
| ------------------------- | --------------------------------------------------------------------------- | ------- | ----------------------------------------------------- |
| `project_name`            | Name of the docker compose project                                          | string  | `foo`                                                 |
| `project_src_folder`      | Source folder containing the project                                        | string  | `foo`                                                 |
| `docker_compose_files`    | List of compose files to deploy                                             | list    | [`docker-compose.yml`, `docker-compose.override.yml`] |
| `project_tmp_dest_folder` | Target folder containing temporary files for the project                    | string  | `/tmp/foo`                                            |
| `copy_project_files_to`   | Target folder to keep files for the project                                 | string  | ``                                                    |
| `docker_host`             | Docker host to use on deployment                                            | string  | `unix:///var/run/docker.sock`                         |
| `exclude_from_templating` | List of path to exclude from jinja templating (file will be copied instead) | list    | `[]`                                                  |

## Differences with `docker_stack_deploy`

This role reuses the same file-staging mechanism as `docker_stack_deploy`, but deploys with plain `docker compose` (via `community.docker.docker_compose_v2`) instead of Docker Swarm (via `community.docker.docker_stack`). No Swarm cluster is required.

Two options from `docker_stack_deploy` have no equivalent here, on purpose:

- **`with_registry_auth`**: on `docker_stack_deploy`, this forwards the caller's registry credentials to every node of the Swarm cluster so each node can pull the image independently. `docker compose` only ever talks to a single daemon (the one pointed to by `docker_host`), so there is no cluster to propagate credentials to. If the target daemon needs to authenticate to pull a private image, that authentication must already be set up on that daemon (e.g. a prior `docker login`, or an existing `~/.docker/config.json`) — this is an infrastructure prerequisite, not something this role manages.
- **`detach`**: `community.docker.docker_compose_v2` always runs `docker compose up` as a non-interactive, scripted call (it starts the project, reads back structured state, and returns) — there is no "attached to the terminal, streaming logs" mode to opt out of, unlike the raw CLI's `-d` flag. The module's behavior is already, in effect, always the detached one.

Not yet exposed (available on `community.docker.docker_compose_v2` but out of scope for this first version, addable later without breaking changes): `pull`, `build`, `wait`/`wait_timeout`, `services`, `scale`, `profiles`, `remove_orphans`.

## Execution modes

### Controller mode (default)

The role runs on the Ansible controller (`hosts: localhost`). Source files are on the controller; `docker_host` points to a remote Docker daemon endpoint.

```yaml
- name: Deploy a test
  hosts: localhost
  roles:
    - name: Deploy test
      role: csm.shared_roles.docker_compose_deploy
      project_name: test
      project_src_folder: templates/test
      project_tmp_dest_folder: "./.tmp/test"
      copy_project_files_to: "/tmp/my-deploy"
      docker_host: "tcp://remote-docker:2376"
      exclude_from_templating:
        - config/file1.txt
      docker_compose_files:
        - docker-compose.yml
        - docker-compose.plus.yml
```

### Target node mode

The role runs on a remote host (the Docker host itself). Source files are still read from the controller; the role templates them directly onto the target. `docker_host` defaults to the local socket.

```yaml
- name: Deploy a test
  hosts: "{{ groups['docker_hosts'][0] }}"
  roles:
    - name: Deploy test
      role: csm.shared_roles.docker_compose_deploy
      project_name: test
      project_src_folder: templates/test       # path on the controller
      project_tmp_dest_folder: "/tmp/test"     # path on the target node
      exclude_from_templating:
        - config/file1.txt
      docker_compose_files:
        - docker-compose.yml
        - docker-compose.plus.yml
      # docker_host defaults to unix:///var/run/docker.sock (local socket on target)
```
