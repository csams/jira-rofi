# jira-rofi
`jira-rofi` contains a script that uses [rofi](https://github.com/davatorium/rofi) to display results from a jira v2 REST API query.  It must
also be run periodically as a background process to populate the cache from which results are displayed.

## Installation
You'll need to have [rofi](https://github.com/davatorium/rofi) installed before beginning.

Create a Personal Access Token (PAT) in your jira instance.  Then create a file at
`$HOME/.config/rofi/jira-rofi-config.json` with the following structure:
```json
{
    "jql": "(assignee=currentUser() or watcher=currentUser()) and status not in (Closed, Resolved) order by updated",
    "url": "https://issues.redhat.com",
    "token": "<your personal access token goes here>"
}
```

The `jql` in the example above is the default if the `jql` key isn't included.  If you populate the cache but
change the `jql`, you'll need to delete the cache file and update it manually to see the new results
immediately.  The bash example below has instructions for manually updating the cache.

The cached data is stored at `$HOME/.cache/rofi/jira-cache.json`

Next, install the python dependencies and the systemd files.  The following stanza likely can't be copied
verbatim, but it gives the idea.
```bash
pip install --user requests filelock

# Ensure jira.rofi is executable and copy it somewhere on your path.  I use $HOME/.local/bin
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
# The cached data will be stored at $HOME/.cache/rofi/jira-cache.json
systemctl --user start jira-rofi.service

# Check the status
systemctl --user status jira-rofi

# Enable the timer.  It will run the service every 15 minutes to update the cache.
# The cached data will be stored at $HOME/.cache/rofi/jira-cache.json
# Logs will go in $HOME/.local/state/jira-rofi/stderr.log
systemctl --user enable jira-rofi.timer
```

## Interactive Mode
You can run the script interactively with `jira.rofi --jql`.  It creates an input loop in which you can run
`jql` queries against the server.  The results are output to the terminal as a table.  `readline` is enabled
with history kept in `$XDG_CACHE_HOME/rofi/jira-rofi-history`.  `$XDG_CACHE_HOME` defaults to `$HOME/.cache`
if unset.

Type any of `exit`, `quit`, `:q`, or `Control-C` to exit the loop.
