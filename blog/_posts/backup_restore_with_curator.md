# Introduction
The best way to manage elasticsearch indices is to use Curator. It ships with both API and CLI tool. For this article we are going to use CLI. Curator has huge list of [feature](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/about-features.html) but we are going to focus on [snapshot](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/snapshot.html) and [restore](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/restore.html).

# Installation
Run following command to install curator.

```
pip install elasticsearch-curator
```

# Setup
Once the curator is installed then we have to specify where the backup will be stored so edit `/etc/elasticsearch/elasticsearch.yml` and add following

```
path.repo: ['/usr/share/elasticsearch/backup']
```

`path.repo` can have any value but the path should be valid otherwise you will get exception.

Next you have to setup the config file for curator that is `curator.yml` By default the curator assumes that the config file is placed at `~/.curator/curator.yml`. But we can ovveride this by passing our custom location to the `--config` option. So for this tutorial i assume that the file is at default location.

So open the `config.yml` and following

```yml
client:
  hosts:
    - "ES_HOST"
  port: "ES_PORT"
  use_ssl: False
  ssl_no_validate: False
  timeout: 30

logging:
  loglevel: CRITICAL
```

> But don't forget to replace `ES_HOST` and `ES_PORT` with your elasticsearch host and port.


Once the `curator.yml` is configured then we have to create a repository. The repository is basically the name of the backup. Therefore in the below command replace `es_snapshot_name` with your backup name and replace `es_snapshot_path` with the path that you assigned earlier to `path.repo`. After replacing the value run the below command.

```
es_repo_mgr create fs --repository es_snapshot_name --location es_snapshot_path --compression true
```

Before moving on to how to take backup and restore part. Lets clear some terminology.

## Action
Actions are the tasks which curator can perform on your indices. These actions will run in sequence and each action is assigned a unique number.

## Options
Options are settings used by actions. For complete list of actions check [following](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/options.html).

## Filters
Filters are the way to select only the indices you want. Check following [link](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filters.html) for more detail.

# Backup
Now lets create `action_backup.yml`. This is the action file that we will use to create snapshop(backup) of our indices. Then add following

```yml
actions:
  1:
    action: snapshot
    description: "Create snapshot"
    options:
      repository: "es_snapshot_name"
      continue_if_exception: False
      wait_for_completion: True
    filters:
      - filtertype: pattern
        kind: regex
        value: ".*$"
```

Now lets tear down the above `action_backup.yml` file. There's only one action and this action is responsible for taking the backup. Replace `es_snapshot_name` with the name that you used earlier while running `es_repo_mgr` command. In the filters i have used filter type [pattern](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filtertype_pattern.html). Which will only get the indicies matching the specified pattern. Now this filter type pattern have different [kinds](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/fe_kind.html). Now the pattern that i specified will take all the indices.

# Restore
Put the following in `action_restore.yml`

```yml
actions:
  1:
    action: close
    description: "Close indices before restoring snapshot"
    options:
      continue_if_exception: True
      ignore_empty_list: True
    filters:
      - filtertype: pattern
        kind: regex
        value: ".*$"
  2:
    action: restore
    description: "Restore all indices in the most recent snapshot with state SUCCESS"
    options:
      repository: "es_snapshot_name"
      name:
      indices:
      wait_for_completion: True
    filters:
      - filtertype: state
        state: SUCCESS
  3:
    action: open
    description: "Open indices after restoring snapshot"
    filters:
      - filtertype: pattern
        kind: regex
        value: ".*$"
```
