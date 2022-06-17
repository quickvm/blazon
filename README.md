# Blazon

A script and systemd unit template for setting up `OnFailure=` notifications on any systemd service unit. It currently supports the following notification channels:

1. Desktop via `notify-send` and [D-Bus](https://www.freedesktop.org/wiki/Software/dbus/)
1. Slack via [Incoming Webhook](https://slack.com/help/articles/115005265063-Incoming-webhooks-for-Slack)

Pagerduty support is being added "soon"™️

## Install the systemd unit template

Automatically:
1. Run `sudo ./install`

Manually:
1. `sudo cp units/* /etc/systemd/system/`
1. `cp bin/blazon /usr/local/bin`
1. Run `systemctl daemon-reload`

## Add Blazon to specific systemd units ✨✨

Add an override for your service on the `blazon@.service` systemd template. In this example we are adding in notifications to [Hashicorp Vault](https://www.vaultproject.io/). Run `systemctl edit blazon@vault.service` and add these overrides to the new unit.

```bash
[Service]
Environment=BLAZON_NOTIFY_DESKTOP=true
Environment=BLAZON_NOTIFY_SLACK=true
Environment=BLAZON_NOTIFY_PAGERDUTY=true
Environment=BLAZON_SLACK_WEBHOOK_URL=https://hooks.slack.com/services/AAAAAAAAAAA/BBBBBBBBBBB/CCCCCCCCCCCCCCCCCCCCCC
```

Next you will add the below to the `vault.service` unit with `systemctl edit vault.service`

```bash
[Unit]
OnFailure=blazon@%N.service
```

Once that is done, reload systemd `systemctl daemon-reload` and tada, you now have notifications on your `vault.service` unit!!🎉

## Add to all systemd units 📣📣

**⚠️ Warning:** This could be noisy if you have a lot of services. Use at your own risk!

Add `OnFailure=blazon@%N.service` to `/etc/systemd/system/service.d/10-blazon.conf`:

```bash
tee /etc/systemd/system/service.d/10-blazon.conf <<EOF
[Unit]
OnFailure=blazon@%N.service
EOF
```

Prevent ensures that if a blazon@.service instance fails it will not trigger an instance named blazon@blazon.service:

```bash
mkdir /etc/systemd/system/blazon@.service.d/
ln -s /dev/null /etc/systemd/system/blazon@.service.d/10-blazon.conf
```

and reload systemd `systemctl daemon-reload`. Now all services will have the `OnFailure=blazon@%N.service` dependency.

## Testing notifications 🧪🧪

Install the `blazon-failure-test.service` unit:

```
install -m 0644 units/blazon-failure-test.service /etc/systemd/system
```

Configure blazon to notify when it fails with `systemctl edit blazon@blazon-failure-test.service` and add:

```bash
[Service]
Environment=BLAZON_NOTIFY_DESKTOP=true
Environment=BLAZON_NOTIFY_SLACK=true
Environment=BLAZON_NOTIFY_PAGERDUTY=true
Environment=BLAZON_SLACK_WEBHOOK_URL=https://hooks.slack.com/services/AAAAAAAAAAA/BBBBBBBBBBB/CCCCCCCCCCCCCCCCCCCCCC
```

Reload systemd with `systemctl daemon-reload` and start the `blazon-failure-test.service` with `systemctl start blazon-failure-test.service`. This unit will fail on start and notify with Blazon. When you are done testing your notifications, you can run the following to remove it:

```bash
systemctl stop blazon-failure-test.service
rm -f /etc/systemd/system/blazon-failure-test.service
rm -rf /etc/systemd/system/blazon@blazon-failure-test.service.d/
systemctl daemon-reload
```

# License

MIT License

Copyright (c) 2022 QuickVM

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
