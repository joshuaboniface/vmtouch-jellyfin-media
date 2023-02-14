# vmtouch-jellyfin-media

This tool is designed to force media files opened with Jellyfin into resident memory using the `vmtouch` utility.

This can have numerous uses:

1. Avoid dying mid-stream if storage is flaky/blocks.

2. Allow faster skipping around the file.

3. Mitigate (some, but very little!) hard drive load.

In my case, #1 (a Ceph cluster that could block reads) is the primary goal.

This script is a really quick and dirty hack. It is specific to my particular install, but I'm providing it as a useful jumping-off point for others who might want to do something similar.

## Prerequisites

* Requires Python 3 with several standard libraries (`subprocess`, `threading`, `signal`, `re`)

* Requires Jellyfin running as a standard operating system process outputting to `journald`.

* Requires the `vmtouch` tool installed and operational.

## Usage

1. Copy the `vmtouch-jellyfin-media` binary to `/usr/local/sbin/`

1. Optionally, adjust the `TOUCH_TYPE` variable if you want to use a non-blocking lock.

1. Copy the `vmtouch-jellyfin-media.service` service unit to `/etc/systemd/system/`

1. Run `sudo systemctl daemon-reload` to reload the systemd configuration

1. Run `sudo systemctl enable --now vmtouch-jellyfin-media` to enable and start the program

It will then continually watch the Jellyfin service output (via `journalctl`) and act when it sees a file opened for playback. Upon opening, the file will be read into memory with `vmtouch` until playback of that file stops, at which time it will release the file from memory.

## Caveats/Limitations

1. Needs `systemd`/`journalctl` in its current iteration, but this could be modified.

1. Requires `root` privileges (for reading the journal, running `vmtouch`, etc.).

1. Only works with locally-opened files; does not work with tools like RFFmpeg.

1. If you stop playback, the file is purged from memory and must be re-read; this was a conscious design decision.
