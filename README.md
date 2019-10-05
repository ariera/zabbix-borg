This is a fork of the borg-zabbix template and scripts of [theranger](https://github.com/theranger/zabbix-borg). I've removed the "non encrypted" monitoring part and the parsable information will only be generated by the backup clients them selfs. I've also moved the shell script stuff to a separate script and use sudo to execute it. That way the zabbix user doesn't have to have the permissions to read the borg repositories it self. I've also added the missing Item and Trigger for the check of the repositories, and a trigger to remind you that zabbix can't find the status information or can't read it.

Zabbix template for monitoring Borg backup repositories. Requires Borg binary to be available in the system being monitored. Monitoring occurs on the backup server.

# Installation on the backup server side
Below is a one-liner to install all at once
Step-By-Step:
1. Copy `/zabbix/zabbix_agentd.d/borg-zabbix.conf` to the Zabbix agent's configuration directory (usually located at `/etc/zabbix`).
2. Copy `/zabbix/scripts/borg_zabbix.sh` to the Zabbix agent's scripts directory (located at `/etc/zabbix/scripts`).
  * You can change the borg repository directory in the 3. line. Defaults to `/data/borg/*/*/`. Keep in mind, the scripts and Zabbix uses the format like that: `/data/borg/<clienthostname>/<backup repo>`. You should rearrange your borg repositories or create symlinks.
3. Copy `/zabbix/sudoers.d/borg_zabbix` to `/etc/sudoers.d` keep in mind: sudo has to be installed before hand.
4. Check the permissions of the files, they should be like follow:
  * `/etc/zabbix/zabbix_agentd.d/borg-zabbix.conf` 0644, owned by root:root
  * `/etc/zabbix/scripts/borg_zabbix.sh` 0755, owned by root:root
  * `/etc/sudoers.d/borg_zabbix` 0440, owned by root:root
5. Import template configuration `templates/borg_zabbix.xml` to Zabbix web frontend.

# Notes
One-liner:
`FILE="/etc/zabbix/zabbix_agentd.d/borg-zabbix.conf" wget -O "${FILE}" https://raw.githubusercontent.com/tschaerni/zabbix-borg/master/zabbix/zabbix_agentd.d/borg-zabbix.conf && chown root:root "${FILE}" && chmod 644 "{$FILE}" ; FILE="/etc/zabbix/scripts/borg_zabbix.sh" mkdir -p /etc/zabbix/scripts && wget -O "${FILE}" https://raw.githubusercontent.com/tschaerni/zabbix-borg/master/zabbix/scripts/borg_zabbix.sh && chown root:root "${FILE}" && chmod 755 "{$FILE}" ; FILE="/etc/sudoers.d/borg_zabbix" wget -O "${FILE}" https://raw.githubusercontent.com/tschaerni/zabbix-borg/master/sudoers.d/borg_zabbix && chown root:root "${FILE}" && chmod 440 "{$FILE}"`

Todo:
1. Maybe add a host makro in zabbix for the location if the repositories

# Encrypted Repositories
For encrypted repositories, a `borg info` command must be run right after the backup and output saved to a file `status.txt` located in the remote repository. Also the output of the `borg check -v` should be appended to `status.txt`. Since those commands are run right after the backup by the client, they will contain the latest status of the repository. This plugin can then reside in the remote host and read the status from `status.txt`. This effectively avoids making encryption keys available to `zabbix` user neither on backup client or remote storage server.

To save the output of aforementioned commands, use something like:
```
	borg info --last 1 user@host:repo | ssh user@host 'cat > repo/status.txt'
	borg check user@host:repo | ssh user@host 'cat >> repo/status.txt'
```
