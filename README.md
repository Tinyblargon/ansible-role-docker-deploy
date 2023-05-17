# Ansible Role: docker deploy

Ansible role to deploy a docker compose stack (file).

## Requirements

### Local system

- `rsync`

### Deployed system

- `docker-ce`
- `docker-ce-cli`
- `containerd.io`

- `docker-compose-plugin` or `docker-compose`

## Role Variables

| **Variable Name**           | **Type**| **Default Value**   | **Description**|
| :---------------------------| :------:| :------------------:| :--------------|
| docker_deploy_source:        | string  | "docker-deploy"     | The local directory containing the docker deployment and compose file. This directory is relative to the main playbooks directory.|
| docker_deploy_destination:   | string  | "/opt/docker-deploy"| The destination folder the deployment should be cloned to.|
| docker_deploy_compose_file:  | string  | "docker-compose.yml"| The name of the docker compose file to bring up.|
| docker_deploy_compatibility: | bool    | true                | Enable the `--compatibility` flag when bringing up the compose file.|
| docker_deploy_remove_orphans:| bool    | true                | Enable the `--remove-orphans` flag when bringing up the compose file.|
| docker_deploy_recreate:      | bool    | false               | Recrate the compose stack when any of the files in `docker_deploy_source:` have changed.|
| docker_deploy_compose_plugin:| bool    | true                | Specify if the `docker compose` or `docker-compose` command should be used, `true` for `docker compose`, `false` for `docker-compose`.|
| docker_deploy_prune:         | bool    | true                | Whether command `docker system prune --all --force` should be executed after a change was made to the compose deployment.|
| docker_deploy_state:         | string  | "present"           | When `"present"` the `docker_deploy_source:` wil be synced to the `docker_deploy_destination:` and the `docker_deploy_compose_file:` will be brought up. When `"absent"` the `docker_deploy_compose_file:` in `docker_deploy_destination:` will be brought down.|
| docker_deploy_absent_volume: | bool    | false                | Only applies when `docker_deploy_state:` is `"absent"`. Enables the --volumes flag when bringing down the compose file, this will delete all volumes that are specified in the compose file. |
| docker_deploy_absent_remove: | bool    | false                | Only applies when `docker_deploy_state:` is `"absent"`. when `true` the `docker_deploy_destination:` will be deleted. |

## Example Playbooks

### Present

```yaml
- hosts: all
  roles:
    - role: tinyblargon.docker_deploy
      vars:
        docker_deploy_source: "docker-deploy"
        docker_deploy_destination: "/opt/docker-deploy"
        docker_deploy_compose_file: "docker-compose.yml"
        docker_deploy_compatibility: true
        docker_deploy_remove_orphans: true
        docker_deploy_recreate: true
        docker_deploy_compose_plugin: false
        docker_deploy_prune: true
        docker_deploy_state: "present"
```

### Absent

```yaml
- hosts: all
  roles:
    - role: tinyblargon.docker_deploy
      vars:
        docker_deploy_destination: "/opt/docker-deploy"
        docker_deploy_compose_file: "docker-compose.yml"
        docker_deploy_compose_plugin: false
        docker_deploy_prune: true
        docker_deploy_state: "absent"
        docker_deploy_absent_volume: true
        docker_deploy_absent_remove: true
```

## License

MIT
