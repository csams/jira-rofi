# jira-rofi
`jira-rofi` displays results from a jira query with [rofi](https://github.com/davatorium/rofi).  It also can be run as a background task to
populate a cache from which results are displayed nearly instantaneously.

## Installation
You'll need to [rofi](https://github.com/davatorium/rofi) installed before beginning.

Create a Personal Access Token (PAT) in your jira instance and copy it as the only line in
`$HOME/.config/rofi/jira-token.txt`.

Next, install the python dependencies and the systemd files.  The following stanza likely can't be copied
verbatim, but it gives the idea.

```bash
pip install --user requests

# Ensure jira.rofi is executable and copy it somewhere on your path. I use $HOME/.local/bin
chmod u+x jira.rofi
cp jira.rofi $HOME/.local/bin

# Copy the systemd files to your user's local systemd config location.
# If you put jira.rofi somewhere other than $HOME/.local/bin, you'll need to update jira-rofi.service to point
# to it.
cp jira-rofi.service $HOME/.config/systemd/user
cp jira-rofi.timer $HOME/.config/systemd/user

# Reload systemd to pick up the new configs
systemctl --user daemon-reload

# Run the service once to build the cache
systemctl --user start jira-rofi.service

# Check the status
systemctl --user status jira-rofi

# Enable the timer. It will run the service every 15 minutes.
# Logs will go in $HOME/.local/state/jira-rofi/stderr.log
systemctl --user enable jira-rofi.timer
```
