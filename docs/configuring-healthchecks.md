<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Alejandro AR
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 MASH project contributors
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Sergio Durigan Junior
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Healthchecks

This is an [Ansible](https://www.ansible.com/) role which installs [Healthchecks](https://healthchecks.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Healthchecks is a cron job monitoring software.

See the project's [documentation](https://healthchecks.io/docs/) to learn what Healthchecks does and why it might be useful to you.

## Prerequisites

To run a Healthchecks instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Healthchecks with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# healthchecks                                                         #
#                                                                      #
########################################################################

healthchecks_enabled: true

########################################################################
#                                                                      #
# /healthchecks                                                        #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Healthchecks you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
healthchecks_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Specify database (optional)

It is necessary to select database used by Healthchecks from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
healthchecks_database_type: postgres
```

Set `mysql` to use a MySQL compatible database and `sqlite` to use SQLite, respectively. The SQLite database is stored in the directory specified with `healthchecks_data_path`.

For other settings, check variables such as `healthchecks_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Configuring connection to database server (optional)

By default the role is configured to establish connection with the database server via the Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the MySQL compatible server's Unix socket path on the host (bind-mount source)
healthchecks_database_mysql_socket_path_host: ""

# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
healthchecks_database_postgres_socket_path_host: ""
```

Setting it enables to connect to the database server via Unix socket mounted in the container.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to the MySQL compatible server via a Unix socket
healthchecks_database_mysql_socket_enabled: false

# Disable the connection to the Postgres server via a Unix socket
healthchecks_database_postgres_socket_enabled: false
```

### Configuring the mailer (optional)

You can configure a SMTP mailer to enable email functions such as notifications. To configure it, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
# Set the hostname of the SMTP server
healthchecks_environment_variable_email_host: ""

# Set the port number of the SMTP server
healthchecks_environment_variable_email_port: 587

# Set the username for the SMTP server
healthchecks_environment_variable_email_host_user: ""

# Set the password for the SMTP server
healthchecks_environment_variable_email_host_password: ""

# Set the email address that emails will be sent from
healthchecks_environment_variable_default_from_email: ""

# Set to `true` to make Healthchecks use TLS encryption
healthchecks_environment_variable_email_use_tls: false

# Set to `true` to skip verification of the TLS certificate on the server
healthchecks_environment_variable_email_use_verification: false
```

>[!WARNING]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. The worst scenario is that your server's IP address or hostname will be included in the spam list such as the one managed by [Spamhaus](https://www.spamhaus.org/). If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Configuring notification services (optional)

On Healthchecks you can add configuration settings of notification services such as ntfy, Gotify, etc. Refer to [this page](https://healthchecks.io/docs/configuring_notifications/) on the official documentation as well about how to configure them.

To set up supported services, refer to the [upstream `.env.example` file](https://github.com/healthchecks/healthchecks/blob/master/docker/.env.example) for environment variables. You can pass those variables to the Healthchecks container with the `healthchecks_environment_variables_additional_variables` variable as below:

```yml
healthchecks_environment_variables_additional_variables: |
  DISCORD_CLIENT_ID=123
  DISCORD_CLIENT_SECRET=456
```

To actually have the services use (and get messages sent through them), you will need to adjust settings on the service's UI after the service is installed.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `healthchecks_environment_variables_additional_variables` variable

See [the official documentation](https://healthchecks.io/docs/self_hosted_configuration/) for a complete list of Healthchecks's config options that you could put in `healthchecks_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Healthchecks becomes available at the specified hostname like `https://example.com`.

To get started, create a superuser account by running the command as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=create-admin-healthchecks -e email=EMAIL_ADDRESS_HERE -e password=PASSWORD_HERE
```

After creating the superuser account, you can open the URL to log in and start setting up monitoring tasks. You can create as many accounts as you wish.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu healthchecks` (or how you/your playbook named the service, e.g. `mash-healthchecks`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
healthchecks_environment_variable_debug: true
```
