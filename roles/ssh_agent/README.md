# Ansible role : ssh_agent

This role allow you to register a ssh key in an agent, get env var for using agent and disabling agent

## ⚙️ Variables

Variables :

| Variable name       | Description                                             | Type   | Default value |
| ------------------- | ------------------------------------------------------- | ------ | ------------- |
| `priv_ssh_key_path` | Path of the ssh key                                     | string |               |
| `existing_agent`    | Environment variables to register in order to use agent | map    |               |
| `state`             | State of the agent                                      | string | `present`     |

## ⚙️ Facts

The role set these facts

| fact name       | Description                                             | Type | Default value |
| --------------- | ------------------------------------------------------- | ---- | ------------- |
| `ssh_agent_env` | Environment variables to register in order to use agent | map  |               |

## Example

### Starting an agent and register a key

```yaml
- name: Register a key
  hosts: localhost
  roles:
    - name: Register a key
      role: csm.shared_roles.ssh_agent
      vars:
          priv_ssh_key_path: ~/.ssh/id_rsa
          state: "present"
```

### Perform tasks using agent
```yaml
- name: task
  hosts: localhost
  gather_facts: false
  environment: "{{ssh_agent_env}}"
  tasks:
    - name: my task
      ... # task will get the environment variables setup for using agent
```

### Closing the agent

```yaml
- name: Closing agent
  hosts: localhost
  roles:
    - name: Register a key
      role: csm.shared_roles.ssh_agent
      vars:
          existing_agent: "{{ssh_agent_env}}"
          state: "absent"
```
