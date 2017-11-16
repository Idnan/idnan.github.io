---
layout: post
title: Backing up and Restoring ES Indices Using Curator
comments: true
---
The best way to manage elasticsearch indices is to use [Curator](https://github.com/elastic/curator). It ships with both API and CLI tool. For this article we are going to use the CLI. Curator has a huge list of [features](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/about-features.html) but we are going to focus on [snapshot](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/snapshot.html) and [restore](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/restore.html) for this article.

# Installation

First things first, lets install the curator. You can easily do that by running the command below

```
pip install elasticsearch-curator
```

# Setting things Up

Once the curator is installed, we need to give it a path where it will store the backups. In order to do that, open up the file `/etc/elasticsearch/elasticsearch.yml` and add following

```
path.repo: ['/usr/share/elasticsearch/backup']
```

Make sure that the path that you set above is valid, otherwise you will get exception.

Next up, we need to setup the config file for curator that is `curator.yml`. By default the curator assumes that the config file is placed at `~/.curator/curator.yml`. But we can override this by passing our custom location using the `--config` option. For now, let's create the file at the default location.

Open the `~/.curator/curator.yml` file and and the below

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

> Make sure to replace `ES_HOST` and `ES_PORT` with your elasticsearch host and port.

Once the `curator.yml` is configured, we need to create a repository. Repository is something that will be holding our backup. In order to do that, run the below command

```
es_repo_mgr create fs --repository es_snapshot_name --location es_snapshot_path --compression true
```

> Make sure to replace `es_snapshot_name` with your backup name and `es_snapshot_path` with the path that you assigned earlier to `path.repo`.

# Backup

Now that we have the setup read, the next step is to create our action file that will be used to create the backup. This file contains the three key parts i.e.

## Action
Actions are the tasks which curator can perform on your indices. These actions will run in sequence and each action is assigned a unique number.

## Options
Options are settings used by actions. For complete list of actions check [following](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/options.html).

## Filters
Filters are the way to select only the indices you want. Check following [link](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filters.html) for more detail.

Here is how our action file looks like

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

Lets tear down the above `action_backup.yml` file. There's only one action and this action is responsible for taking the backup. Replace `es_snapshot_name` with the name that you used earlier while running `es_repo_mgr` command. In the filters, I have used the filter type [`pattern`](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filtertype_pattern.html) which will only get the indicies matching the specified pattern. Now this filter type pattern have different [kinds](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/fe_kind.html). The kind that we have specified here is `regex` and since we want to match all the indices, we are going to use the value of `.*$`

Once the `action_backup.yml` is ready then run below command to take backup

```
curator action_backup.yml
```

# Restore

In order to restore our backup, put the below in `action_restore.yml`

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

As you can see, the restore action file has 3 actions. First action with number `1` is for closing the indices so that we can halt any actions that are being performed in order to proceed with the restore. In task numbered `2` we are doing the actual restore and in task `3` we are re-opening the closed indices so that we can use them for search.

Now our  backup file is ready, so use following command to restore the backup
```
curator action_restore.yml
```

And that about wraps it up. If you would like to learn more, [find the original docs here](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html). Feel free to leave your comments and feedback down below.
