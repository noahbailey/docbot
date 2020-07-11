---
title: "SSH Agent"
---

# SSH Agent

Create a systemd user-level config directory:

       mdkir -p .config/systemd/user/

Create the service unit file:

`.config/systemd/user/ssh-agent.service`

	[Unit]
	Description=SSH key agent

	[Service]
	Type=simple
	Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
	ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK

	[Install]
	WantedBy=default.target

Enable the unit.

       systemctl --user enable --now ssh-agent

Add this to `.bashrc` so that the shell can find the socket:

       export SSH_AUTH_SOCK="${XDG_RUNTIME_DIR}/ssh-agent.socket"

Then, add a key to the agent:

      ssh-add ~/.ssh/your-key.pem

From now until logout, keys will be stored in the agent.

Sensitive keys can be removed after use with `ssh-add -D`


### References

* https://unix.stackexchange.com/questions/339840/how-to-start-and-use-ssh-agent-as-systemd-service
* https://wiki.archlinux.org/index.php/SSH_keys#Start_ssh-agent_with_systemd_user
