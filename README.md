# status-msg-teams

Utility for reporting SystemD errors to Microsoft Teams.

## Setup

1. Install dependencies:

   * jq
   * wget

   ```sh
   sudo dnf install jq
   ```
2. Clone this repository on the server in question. We assume it is located at
   `/srv/status-msg-teams` in the following instructions.
3. Make `send-status-to-teams` available to run:

   ```sh
   sudo ln -s /srv/status-msg-teams/send-status-to-teams /usr/local/bin/send-status-to-teams
   ```
4. Copy in the SystemD service file:

   ```sh
   sudo cp status-msg-teams@.service /etc/systemd/system
   ```
5. Edit the file (e.g. `sudo nano
   /etc/systemd/system/status-msg-teams@.service`) and replace `<URL>` with a
   [webhook URL acquired from Microsoft Teams][ms webhook].
6. Make SystemD aware of our changes:

   ```sh
   sudo systemctl daemon-reload
   ```

[ms webhook]: https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/connectors/connectors-using#setting-up-a-custom-incoming-webhook


## Usage

This section describes the ways this utility can be used.


### As shell script

`send-status-to-teams` is available as a command you can run.

Usage:

```sh
send-status-to-teams unit [webhook-URL]
```


#### Parameters

**`unit`**: the name of the SystemD unit you would like to report the status of.

**`webhook-URL`**: the Microsoft Teams webhook the status message
should be sent to. If this parameter is not given, the [environment variable]
called `WEBHOOK_URL` will be used instead.

[environment variable]: https://en.wikipedia.org/wiki/Environment_variable


#### Behaviour

When run, a message will be posted to the provided webhook URL. The message is
the same as the output provided by `systemctl status UNIT`, with an appropriate
title.


### As SystemD service

This is a so-called parameterized SystemD service. This means that it is a template, which you can create many new services out of.

The parameter is the name of the SystemD unit you would like to report the status of. You place it after the `@`, but before `.service` when referring to the service using `systemctl` and so on.

Say you have a unit named `example.service`. The following command immediately reports its status to Microsoft Teams:

```sh
sudo systemctl start status-msg-teams@example.service.service
#                                     ^^^^^^^^^^^^^^^ <- parameter
```

In the example above, `example.service` is the parameter. The second `.service` applies to the entire name (including `status-msg-teams`). You can think of it as `(status-msg-teams@(example.service)).service`.


### As automatic report when SystemD service fails

Just like you can invoke the service manually with `systemctl`, SystemD can invoke the service automatically when certain conditions are met. To report the status when an error occurs, you can set the [OnFailure directive] to `status-msg-teams@%n.service`. `%n` is automatically replaced with the unit's name.

For example, for `example.service`:

```systemd
[Unit]
Description=Example service
OnFailure=status-msg-teams@%n.service

[Service]
Type=oneshot
ExecStart=/usr/bin/echo Oh wow, what an example
```

[OnFailure directive]: https://www.freedesktop.org/software/systemd/man/systemd.unit.html#OnFailure=


## Links

* [Inspiration for this utility](https://wiki.archlinux.org/index.php/Systemd/Timers#MAILTO)
