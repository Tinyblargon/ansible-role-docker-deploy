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

| **Variable Name**            | **Type**| **Default Value**        | **Description**|
| :----------------------------| :------:| :-----------------------:| :--------------|
| docker_deploy_source:        | string  | "docker-deploy"          | The local directory containing the docker deployment and compose file. This directory is relative to the main playbooks directory.|
| docker_deploy_destination:   | string  | "/opt/docker-deploy"     | The destination folder the deployment should be cloned to.|
| docker_deploy_compose_file:  | string  | "docker-compose.yml"     | The name of the docker compose file to bring up.|
| docker_deploy_files:         | list map| []                       | A list of files that should be copied to the `docker_deploy_destination:`, more information about the map can be found in the following chapter [Docker_deploy_files](#docker_deploy_files).|
| docker_deploy_compatibility: | bool    | true                     | Enable the `--compatibility` flag when bringing up the compose file.|
| docker_deploy_remove_orphans:| bool    | true                     | Enable the `--remove-orphans` flag when bringing up the compose file.|
| docker_deploy_build:         | bool    | false                    | Enable the `--build` flag when any of the files in `docker_deploy_source:` or `docker_deploy_files:` have changed.|
| docker_deploy_recreate:      | bool    | false                    | Enable the `--force-recreate` flag when any of the files in `docker_deploy_source:` or `docker_deploy_files:` have changed.|
| docker_deploy_compose_plugin:| bool    | true                     | Specify if the `docker compose` or `docker-compose` command should be used, `true` for `docker compose`, `false` for `docker-compose`.|
| docker_deploy_prune:         | list    | ["image"]                | Which prune commands should be executed after a change was made to the compose deployment. The value can be a combination of any of the following options `"builder"`, `"container"`, `"image"`, `"network"`, `"system"`, `"volume"`.|
| docker_deploy_state:         | string  | "present"                | When `"present"` the `docker_deploy_source:` wil be synced to the `docker_deploy_destination:` and the `docker_deploy_compose_file:` will be brought up. When `"absent"` the `docker_deploy_compose_file:` in `docker_deploy_destination:` will be brought down.|
| docker_deploy_rsync_opts:    | list    | ['--exclude="*.example"']| Specify additional rsync options by passing in an array. All `dest` specified in`docker_deploy_files:` will automatically be excluded from rsync.|
| docker_deploy_show_warnings: | bool    | true                     | Show the docker compose warnings. |
| docker_deploy_absent_volume: | bool    | false                    | Only applies when `docker_deploy_state:` is `"absent"`. Enables the --volumes flag when bringing down the compose file, this will delete all volumes that are specified in the compose file. |
| docker_deploy_absent_remove: | bool    | false                    | Only applies when `docker_deploy_state:` is `"absent"`. when `true` the `docker_deploy_destination:` will be deleted. |

### Docker_deploy_files

| **Variable Name**| **Type**| **Default Value**| **Description**|
| :----------------| :------:| :---------------:| :--------------|
| content:         | string  | ""               | When used instead of `src`, sets the contents of a file directly to the specified value. Works only when `dest` is a file. Creates the file if it does not exist.|
| dest:            | string  |                  | Remote path relative to the `docker_deploy_destination:` where the file should be copied to. If `src` is a directory, this must be a directory too. If `dest` is a non-existent path and if either dest ends with “/” or `src` is a directory, `dest` is created. If `dest` is a relative path, the starting directory is determined by the remote host. If `src` and `dest` are files, the parent directory of `dest` is not created and the task fails if it does not already exist. All `dest` will automatically be excluded from rsync.|
| group:           | string  | "root"           | Name of the group that should own the filesystem object, as would be fed to chown.|
| mode:            | string  |                  | The permissions of the destination file or directory. [Ansible documentation for further details](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#parameter-mode) |
| owner:           | string  | "root"           | Name of the user that should own the filesystem object, as would be fed to chown. Specifying a numeric username will be assumed to be a user ID and not a username. Avoid numeric usernames to avoid this confusion.|
| src:             | string  |                  | Local path to a file to copy to the remote server. This can be absolute or relative. If path is a directory, it is copied recursively. In this case, if path ends with “/”, only inside contents of that directory are copied to destination. Otherwise, if it does not end with “/”, the directory itself with all contents is copied. This behavior is similar to the rsync command line tool.|
| no_log:          | bool    | false            | If `true`, the content can not be logged.|

When `content:` and `src:` are both unspecified, an empty file is created at the destination.

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
        docker_deploy_files:
          - dest: ".env"
            src: ".env"
          - dest: ".secrets/password.txt"
            content: "{{ lookup('file', 'password.txt') }}"
            owner: "root"
            group: "root"
            mode: "0600"
            no_log: true
        docker_deploy_compatibility: true
        docker_deploy_remove_orphans: true
        docker_deploy_recreate: true
        docker_deploy_compose_plugin: false
        docker_deploy_prune: ["system"]
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
        docker_deploy_prune: ["system", "volume"]
        docker_deploy_state: "absent"
        docker_deploy_absent_volume: true
        docker_deploy_absent_remove: true
```

## License

MIT
