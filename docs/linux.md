# Linux

Tips, tricks, and tutorials for using Linux and its various distributions.

## Quick Reference

### Linux Permissions

Linux permissions are represented by a 10-character string where the first character indicates the type of file (whether it is a file, directory, symbolic link, etc.) and the next 9 characters represent the permissions for the owner, group, and others, respectively. In Linux, permissions are set for three types of users: the owner of the file/directory, the members of the file/directory's group, and all other users (also known as "world" or "others"). Each of the types of users have 3 kinds of permissions: read, write and execute. Below is a table that explains what each character in a Linux permission string represent:

| Number | Permission Type          | Symbol |
|--------|--------------------------|--------|
| 0      | No Permission            | ---    |
| 1      | Execute                  | --x    |
| 2      | Write                    | -w-    |
| 3      | Execute + Write          | -wx    |
| 4      | Read                     | r--    |
| 5      | Read + Execute           | r-x    |
| 6      | Read + Write             | rw-    |
| 7      | Read + Write + Execute   | rwx    |

The numbers are another way to represent the same permission as an octal representation of the string. For instance, `0740` would be `drwxr-----` where `7` is `rwx`, `4` is `r--` and `0` is `---`.

As stated above, the first character indicates the type of file. This character is also known as the file mode or file type indicator. The file type indicator shows whether the file is a regular file, directory, symbolic link, socket, pipe, or other types of files.

The most common file types and their corresponding file type indicators are:

- `-` (a dash) for a regular file
- `d` for a directory
- `l` for a symbolic link
- `s` for a socket
- `p` for a named pipe (FIFO)
- `c` for a character device file
- `b` for a block device file

For example, in the Linux file permission string `-rw-r--r--`, the first character `-` indicates that the file is a regular file. If the file were a directory, the first character would be `d` instead of `-`. The remaining `rw-r--r--` can be divided into 3 parts:

- Owner: `rw-`
- Group: `r--`
- Others: `r--`

So in this example, the owner can read and write but the group and the others can only read.

### SSH Folder/File Permissions

The `.ssh` folder is a directory used to store the SSH keys on a Linux system. The default permissions for this folder are usually set to `700`, which means that only the owner of the folder has full read, write, and execute permissions, while members of the group and other users have no permissions at all. This is done to ensure that the SSH keys are kept secure and not accessible by other users on the system.

However, Linux provides a warning about the permissions on the `.ssh` folder because if the permissions are set too loosely (i.e. `755`), it can make the SSH keys and configuration files vulnerable to theft or misuse by unauthorized users. Therefore, it is important to ensure that the permissions on the .ssh folder are set correctly and that only the authorized users have access to it. It is also recommended to use encryption and passphrases for SSH keys to provide an extra layer of security.

| Folder | Permission (octal) | Permission (string) |
|--------------|--------|--------------|
|`.ssh` folder | `0700` | `drwx------` |
|`authorized_keys` | `0600` | `-rw-------` |
| `id_rsa` (Private key) | `0600` | `-rw-------` |
| `id_rsa.pub` (Public key) | `0644` | `-rw-r--r--` |
| `known_hosts` | `0600` | `-rw-------` |

## Common Issues and Solutions

### Booting Debian Installer over Serial

Navigate to Install instead of Graphical Install `TAB` (or `H` depending on version) when at the installer screen and use the following parameters (replace with your intended device, baud-rate and data bits), and then hit `ENTER`:

```bash
install console=ttyS0,115200n8
```

### Fixing "Key is stored in legacy trusted.gpg keyring" Issue

If you are getting the following error when trying to install packages (e.g. Slack):

```bash
W: https://packagecloud.io/slacktechnologies/slack/debian/dists/jessie/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

You can fix it by running:

```bash
sudo apt-key list
```

This will show a list of keys stored in your system. Look for the key that is associated with the repository causing the warning message. In the case of Slack, it is `https://packagecloud.io/slacktechnologies/slack/debian/dists/jessie/InRelease`.

```bash
--------------------
pub   rsa4096 2014-01-13 [SCEA] [expired: 2019-01-12]
      418A 7F2F B0E1 E6E7 EABF  6FE8 C2E7 3424 D590 97AB
uid           [ expired] packagecloud ops (production key) <ops@packagecloud.io>

pub   rsa4096 2016-02-18 [SCEA]
      DB08 5A08 CA13 B8AC B917  E0F6 D938 EC0D 0386 51BD
uid           [ unknown] https://packagecloud.io/slacktechnologies/slack (https://packagecloud.io/docs#gpg_signing) <support@packagecloud.io>
sub   rsa4096 2016-02-18 [SEA]
```

We need to take the last 8 characters (excluding the space) under the line after pub. If there are multiple pub lines, you can use the one that is not expired. In this case, it is `0386 51BD`.

Now, run the following command to add the GPG key to `/etc/apt/trusted.gpg.d` replacing `<key>` with the key you found in the previous step and `<name>` with the name of the repository (e.g. slack):

```bash
sudo apt-key export <key> | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/<name>.gpg
```

Example:

```bash
sudo apt-key export 038651BD | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/slack.gpg
```

You can ignore the error message:

```bash
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
```

Now, you should be able to run `apt update` and install packages from the repository without getting the warning message.

Credit: Abhishek Prakash, Jan 2023 ([https://itsfoss.com/key-is-stored-in-legacy-trusted-gpg/](https://itsfoss.com/key-is-stored-in-legacy-trusted-gpg/))

### Debian APT Mirrors

I ran into a strange issue where Debian kept trying to pull from mirrors I never specified like `debian.gtisc.gatech.edu`. Turns out, there is another file where mirrors are stored:

```bash
/usr/share/python-apt/templates/Debian.mirrors
```

Getting rid of the line with `debian.gtisc.gatech.edu` made the issue happen less frequently, but it looks like Debian sometimes tries to pull from `debian.gtisc.gatech.edu` anyway. This is a complaint I've seen from other people as well.

### Fix Broken resolv.conf

If you are having issues with DNS resolution, you can try to fix it by running:

!!! warning
    This will only work if you are using systemd-resolved. It will also wipe out any custom DNS servers you have set manually in `/etc/resolv.conf`.

```bash
sudo rm /etc/resolv.conf
sudo ln -sv /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### Fedora upadte-grub alternative

For whatever reason, Fedora doesn't have an `update-grub` command, so here is the alternative:

```bash
sudo grub2-mkconfig -o $(readlink -f /etc/grub2-efi.cfg)
```

### Pipewire GNOME Audio Slider/Hotkeys Not Working

If you are using Pipewire and the Gnome audio slider/hotkeys iare not working, you can try to fix it by running:

```bash
systemctl restart --user pipewire pipewire-session-manager
```

Something that may or may not have helped me was:

```bash
sudo nano /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf
```

and setting `wifi.powersave = 3` to `wifi.powersave = 2`.

I also changed the Bluetooth HID timeout:

```bash
sudo nano /etc/bluetooth/input.conf
```

and set `IdleTimeout=0`. Then restart `pipewire` and `pipewire-session-manager again`. I am unsure of which of these things fixed the issue (if any of them did).

## Restart Broken KDE Plasma Shell

Sometimes (especially on the Debian testing branch w/ Nvidia drivers) the KDE Plasma Shell just goes haywire. This is my go-to:

```bash
kquitapp5 plasmashell && kstart5 plasmashell &
```

followed by a `disown`. Every now and then, presumably because of how the KDE Plasma desktop is D-Bus enabled, I need to ommit the `&` and instead do `CTRL-Z` followed by the `bg` and `disown` commands. More on that [here](#linux-disown).

## Alpine Not Automounting fstab

Simply run this command:

```shell
rc-update add netmount boot
```

## Raspberry Pi Headless SSH Setup

1. Create an empty file named `ssh` in the boot partition of the SD card.

    ```bash
    touch (location to boot partition)/ssh
    ```

2. Configure SSH password for new user. Prior to late 2022, the default user/pass was `pi/raspberry`. Now, there is no default password. Instead, we need to make a file called `userconf`. We then either need to generate a password or use the old default one.

    ```bash
    sudo nano (location to boot partition)/userconf
    ```

    If you want to generate a password, run:

    ```bash
    echo 'yourpassword' | openssl passwd -6 -stdin
    ```

    This will output a hash that you can use in the `userconf` file. The `userconf` file should look like this:

    ```bash
    pi:yourpasswordhash
    ```

    If you want to use the old default password, the `userconf` file should look like this:

    ```bash
    pi:$6$6jHfJHU59JxxUfOS$k9natRNnu0AaeS/S9/IeVgSkwkYAjwJfGuYfnwsUoBxlNocOn.5yIdLRdSeHRiw8EWbbfwNSgx9/vUhu0NqF50
    ```

3. Insert the SD card into the Raspberry Pi and boot it up.

4. Connect to the Raspberry Pi via SSH. The default hostname is `raspberrypi`.

## Install Linux VM on Android

[https://github.com/sylirre/vmConsole](https://github.com/sylirre/vmConsole)

## Useful Commands

### Linux Disown

The `disown` command is used to remove a job from the current shell's list of jobs. This is useful if you want to run a command in the background and then exit the shell without killing the process.

#### Disown Running Application

Here is an example of how to use `disown`:

1. Start a long-running command in the terminal, such as:

    ```bash
    sleep 1000 &
    ```

    This will start a `sleep` command that will run for 1000 seconds (a little over 16 minutes).

2. Press `CTRL-Z` to pause the command and return to the shell prompt.

3. Run the `jobs` command to see the list of jobs that are currently running or paused:

    ```bash
    $ jobs
        [1]+  Stopped                 sleep 1000`
    ```

    The output shows that there is one job running (`sleep 1000`) and it is currently stopped (paused).

4. Run the `disown` command followed by the job number (which is `1` in this case):

    ```bash
    disown %1
    ```

    This will detach the `sleep` command from the shell session and prevent it from being terminated when the shell session is closed.

5. To confirm that the job is no longer being tracked by the shell, run the `jobs` command again:

    ```bash
    jobs
    ```

    There should be no output because there are no jobs being tracked by the shell.

At this point, you can safely close the terminal or log out of the system without worrying about the `sleep` command being terminated. The `sleep` command will continue running in the background until it completes or is terminated by some other means.

#### Find Running Application After Linux Disown

To find a running application that has been disowned in Linux, you can use the `pgrep` command along with the name of the application.

Here is an example command:

```bash
pgrep <application_name>
```

Replace `<application_name>` with the name of the application you want to find. This command will return the process ID (PID) of the application if it is currently running.

Alternatively, you can use the `ps` command to find the running application along with its associated processes. Here is an example command:

```bash
ps aux | grep <application_name>
```

Replace `<application_name>` with the name of the application you want to find. This command will display a list of processes that match the specified application name. The first column of the output will be the PID of the running application.

### Removing Files Matching a Specific Pattern Using `grep` and `rm`

Sometimes, you may need to search for and remove files that match a specific pattern or contain a particular text string. You can achieve this using a combination of the `grep` and `rm` commands in a Unix-like environment.

##### Command Explanation

Here's a breakdown of the command:

- **`grep -rl "NT_STATUS_OBJECT_NAME_NOT_FOUND" .`**: This part of the command uses `grep` to recursively search for files (`-r`) in the current directory (`.`) that contain the text string "NT_STATUS_OBJECT_NAME_NOT_FOUND." The `-l` option tells `grep` to only list the names of matching files.

- **`|` (Pipe)**: The pipe symbol (`|`) is used to pass the list of matching file names as input to the next part of the command.

- **`while IFS= read -r file; do rm -f "$file"; done`**: This part of the command sets up a loop to process each matching file. Here's what each component does:
  - `while IFS= read -r file;`: This initiates a loop that reads each file name (line) from the list of matching files.
  - `do`: Indicates the start of the loop's commands.
  - `rm -f "$file";`: Within the loop, it uses the `rm` (remove) command to forcefully delete (`-f`) the file specified by the `$file` variable, which represents each matching file.
  - `done`: Marks the end of the loop.

##### Result

Executing this command will search for all files in the current directory and its subdirectories that contain the text "NT_STATUS_OBJECT_NAME_NOT_FOUND." For each matching file, it will remove the file using the `rm` command with the `-f` option to force removal.

This approach is not limited to specific patterns or text strings and can be adapted for various use cases where you need to search for and remove files based on specific criteria.

!!! warning "Caution"
    Be extremely careful when using the `rm` command with the `-f` option, as it forcefully deletes files without confirmation, and there is no easy way to recover them. Make sure you are certain about the files you want to remove, and use this command responsibly.

### Making Filenames Windows Friendly with `detox`

To make all of the filenames in a directory Windows-friendly, you can use the `detox` command. This tool helps you clean and sanitize filenames, ensuring compatibility with Windows file systems.

```shell
detox -r .
```

### Removing the Last Character from Filenames with rename

If you need to remove the last character from the filename of every file in the current directory, you can use the `rename` command with a regular expression.

```shell
rename "s/.{1}$//" *
```

### Recursively Checking Files for a String with grep

To recursively check files for a specific string within a directory, you can use the `grep` command with the `-Rnw` options.

```shell
grep -Rnw '/path/to/somewhere/' -e 'pattern'
```

### Prettifying and Scrolling Through JSON

If you want to prettify and scroll through JSON data while highlighting syntax in color, you can use `jq` in conjunction with `less`:

```shell
jq -C . [jsonfile] | less -r
```

## Useful Applications

### Graphical Applications

#### KDE Gnome-like Screenshots

If you are using KDE and want to take screenshots like you would in Gnome (i.e. select a region to screenshot and it will automatically copy to clipboard), you can install `flameshot` and then set the following in System Settings > Workspace > Shortcuts > Add Command:

I used the following command: `flameshot gui -c` for `Take Screenshot (Copy to Clipboard)`

You can also use the following commands:

##### Save to File

- `flameshot gui -p ~/Pictures/Screenshots` for `Take Screenshot`
- `flameshot full -p ~/Pictures/Screenshots` for `Take Screenshot (Full Screen)`
- `flameshot screen -p ~/Pictures/Screenshots` for `Take Screenshot (Current Screen)`

##### Copy to Clipboard

- `flameshot gui -c` for `Take Screenshot (Copy to Clipboard)`
- `flameshot full -c` for `Take Screenshot (Full Screen) (Copy to Clipboard)`
- `flameshot screen -c` for `Take Screenshot (Current Screen) (Copy to Clipboard)`

### Command-Line Applications

#### thefuck

`thefuck` is a command-line tool that corrects your previous command. Think of it as a CLI `auto-correct` feature. It works by analyzing the output of the previous command and then running the correct command based on the output.

GitHub Link: [https://github.com/nvbn/thefuck](https://github.com/nvbn/thefuck)

#### zoxide

`zoxide` is a command-line tool that helps you navigate your filesystem. It keeps track of the directories you use the most and makes it easy to jump to them from anywhere. It is essentially a smarter `cd` command.

GitHub Link: [https://github.com/ajeetdsouza/zoxide](https://github.com/ajeetdsouza/zoxide)

#### fzf

`fzf` is a command-line fuzzy finder. It can be used to search through files and run commands. It is similar to `Ctrl+R` in Bash, but it is much more powerful. It can also be used to search through the output of other commands.

GitHub Link: [https://github.com/junegunn/fzf](https://github.com/junegunn/fzf)

#### bat

`bat` is a `cat` clone with syntax highlighting and Git integration. It is similar to `cat` but it is much more powerful. It can be used to view files and it can also be used to search through files. It is written in Rust which leads to it being very performant.

GitHub Link: [https://github.com/sharkdp/bat](https://github.com/sharkdp/bat)

#### tldr

`tldr` is a collection of community-maintained pages similar to that of man pages, but much simpler. It is a great alternative to `man` pages because it is much easier to read and it is more concise. It's colorfully formatted and it is easy to navigate.

GitHub Link: [https://github.com/tldr-pages/tldr](https://github.com/tldr-pages/tldr)

#### scc

`scc` is a tool that counts lines of code in a directory. It can be used to quickly get a rough estimate of how many lines of code are in a project. It also shows cost to develop and complexity.

GitHub Link: [https://github.com/boyter/scc](https://github.com/boyter/scc)

#### exa

`exa` is a replacement for `ls` that is written in Rust. It can display file-type icons, colors, file/folder info and has several output formats - tree, grid or list. It is also much more colorful and it is easier to read.

GitHub Link: [https://github.com/ogham/exa](https://github.com/ogham/exa)

#### duf

`duf` is a disk usage/du utility for showing info about mounted disks. It is similar to `du` but it is more colorful and it is easier to read. It also has a progress bar that shows how much space is used.

GitHub Link: [https://github.com/muesli/duf](https://github.com/muesli/duf)

## Credits

- [CLI tools you won't be able to live without, Alicia Sykes](https://dev.to/lissy93/cli-tools-you-cant-live-without-57f6)
