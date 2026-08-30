# CLI Cheatsheet

## Common

### Git

#### git clone
- Examples:
    - Clone repo: `git clone <RepoUrl> "<TargetDir>"`
- Notes:
    - `<TargetDir>` is optional; if omitted, clones into a new folder in the current directory named after the repo
- Flags:
    - `--branch[-b] <branchName>`: clone and checkout specific branch

---

#### git status
- Examples:
    - Show working tree status: `git status`

---

#### git add
- Examples:
    - Stage specific file: `git add "<path>"`
    - Stage all changes: `git add --all`
- Flags:
    - `--all[-A]`: stage all changes (new, modified, deleted) across the repo

---

#### git commit

- Examples:
    - Commit: `git commit --message "<message>"`

---

#### git checkout

- Examples:
    - Create local branch: `git checkout -b <branchName>`
    - Switch to branch: `git checkout <branchName>`

---

#### git branch
- Examples:
    - List branches: `git branch --all`
    - Delete branch: `git branch --delete <branchName>`
    - Rename current branch: `git branch --move <newName>`
- Flags:
    - `--all[-a]`: list local and remote-tracking branches
    - `--delete[-d]`: delete branch (safe, must already be merged)
    - `--delete --force[-D]`: force delete unmerged branch
    - `--move[-m]`: rename branch

---

#### git merge

- Examples:
    - Merge <branch1> into <branch2>: `git checkout <branch2> && git merge <branch1>`
    - Abort merge: `git merge --abort`
- Flags:
    - `--abort`: abort a merge in progress due to conflicts

---

#### git stash
- Examples:
    - Stash changes: `git stash push --message "<message>"`
    - List stashes: `git stash list`
    - Apply latest stash: `git stash pop`
    - Apply specific stash: `git stash pop stash@{<index>}`
- Notes:
    - Use `git stash list` to find the index of the stash to target, then apply/pop by `stash@{<index>}`
- Flags:
    - `push --message[-m] "<message>"`: save current changes with a label
    - `pop`: apply and remove the latest stash
    - `apply`: apply latest stash without removing it
    - `drop`: delete a stash without applying it

---

#### git remote
- Examples:
    - List remotes: `git remote -v`

---

#### git pull
- Examples:
    - Pull with rebase: `git pull --rebase`
- Flags:
    - `--rebase`: reapply local commits on top of fetched branch instead of merging

---

#### git push

- Examples:
    - Push local branch to remote: `git push --set-upstream origin <branchName>`
- Notes:
    - `HEAD` can be used in place of `<branchName>` to push the currently checked out branch

---

### Media

#### ffmpeg (Trim Video)

- Examples:
    - Trim without re-encoding: `ffmpeg -ss <StartTimestamp> -to <EndTimestamp> -i "<InputVideoPath>" -c copy "<OutputVideoPath>"`
- Notes:
    - Timestamps must be in format `HH:MM:SS.mmm`
    - `-c copy` is shorthand for `-vcodec copy -acodec copy` - copies both video and audio codec

---

#### ffmpeg (Extract Frame at Timestamp)

- Examples:
    - Extract frame: `ffmpeg -ss <Timestamp> -i "<InputVideoPath>" -vframes <FrameCount> "<output_frame_%03d.jpg>"`
- Notes:
    - Timestamp is the start point from which frames are captured, format `HH:MM:SS.mmm`
- Flags:
    - `-vframes <count>`: number of frames to capture from start timestamp
    - `-s <Width>x<Height>`: pixel size of each captured frame

---

#### ffmpeg (Change Video Container)

- Examples:
    - Remux without re-encoding: `ffmpeg -i "<InputVideoPath>" -c copy "<OutputVideoPath>"`
- Notes:
    - `-c copy` copies both video and audio encodings to the new container

---

#### ffmpeg (Copy Video Without Metadata)

- Examples:
    - Strip metadata: `ffmpeg -i "<InputVideoPath>" -map_metadata -1 -c copy "<OutputVideoPath>"`

---

#### ffmpeg (Download M3U8 Stream)

- Examples:
    - Download stream: `ffmpeg -i "<url.m3u8>" -c copy "<OutputVideoPath>"`
- Notes:
    - Downloads and combines stream without re-encoding
    - When a playlist URL is provided, quality is auto-selected and may give poor viewing experience - use `yt-dlp` instead for quality control

---

#### yt-dlp (Download M3U8 Stream)

- Examples:
    - Download stream: `yt-dlp "<url.m3u8>" --output "<OutputVideoPath>"`
- Notes:
    - Selects highest quality stream by default
- Flags:
    - `--output`: specify output path
    - `--no-mtime`: set modified timestamp to current datetime instead of what server provides

---

#### ffmpeg (Video Transformations)

- Examples:
    - GPU Nvidia: `ffmpeg -hwaccel cuda -i "<InputVideoPath>" -vf "scale=<Width>:<Height>,hflip" -c:v h264_nvenc -rc:v vbr -qmin:v 19 -qmax:v 25 -c:a copy "<OutputVideoPath>"`
    
        h264_nvenc specific flags:
        - `-hwaccel cuda`: enable GPU-accelerated decoding via CUDA
        - `-rc:v vbr`: rate control mode; required to use `-qmin:v` / `-qmax:v`;  vbr in our case is recommended since it varies bitrate when its needed to achieve a desired quality level.
        - `-qmin:v <0-51>`: Nvidia best allowed quality; 19 is visually lossless
        - `-qmax:v <0-51>`: Nvidia worst allowed quality; 25 is a good balance
    - GPU Intel: `ffmpeg -hwaccel qsv -hwaccel_output_format qsv -i "<InputVideoPath>" -vf "scale=<Width>:<Height>,hflip" -c:v h264_qsv -global_quality 22 -c:a copy "<OutputVideoPath>"`
        
        h264_csv specific flags:
        - `-hwaccel qsv`: enable GPU-accelerated decoding via Intel QSV
        - `-global_quality <0-51>`: encoding quality; lower = better; 22 is a good balance
    - CPU: `ffmpeg -i "<InputVideoPath>" -vf "scale=<Width>:<Height>,hflip" -c:v libx264 -crf 23 -c:a copy "<OutputVideoPath>"`
        
        libx264 specific flags
        - `-crf <0-51>`: encoding quality; lower = better; 20 is a good balance for quality and size
- Notes:
    - `-vsync vfr`: add after input source when encountering variable framerate videos (requires CPU encoder)
- Flags:
    - `-c:a copy`: copy audio stream as-is
    - `-c:v <encoder>`: choose video encoder; GPU encoders are fast but less format-compatible; `libx264` (CPU) is broadly compatible
    - `-vf "<filter1>,<filter2>"`: apply one or more video filters
        - `scale=<X>:<Y>`: resize video
        - `hflip`: flip video horizontally
    - `-r <fps>`: convert video to desired framerate

---

#### ffmpeg (Concatenate Videos Losslessly)

- Examples:
    - Concat: 
      ```shell
      tmp=".ffmpeg.concat.tmp"
      printf "file '%s'\n" "<InputPath1.mp4>" "<InputPath2.mp4>" > "$tmp"
      ffmpeg -f concat -safe 0 -i "$tmp" -c copy "<OutputPath.mp4>"
      rm "$tmp"
      ```
- Notes:
    - Requires inputs with matching codecs/parameters
    - `-safe 0` allows absolute/relative paths with special characters in the concat list

---

#### ffmpeg (Check Video for Corruption)

- Examples:
    - Check: `ffmpeg -v error -i "<video.mp4>" -f null -`
- Notes:
    - If any output is produced, the video may be corrupt

---

#### exiftool (File/Media Metadata)

- Examples:
    - List all metadata: `exiftool -api LargeFileSupport=1 -extractEmbedded -G "<FileOrFolderPath>"`
    - Read tag:          `exiftool -api LargeFileSupport=1 -extractEmbedded -G -<TagName> "<FileOrFolderPath>"`
    - Write tag:         `exiftool -<TagName>=<value> "<FileOrFolderPath>"`
    - Delete tag:        `exiftool -<TagName>= "<FileOrFolderPath>"`
- Flags:
    - `-api LargeFileSupport=1`: support larger files; no reason to omit
    - `-extractEmbedded`: extract tags from embedded files as well
    - `-G`: prefix each tag line with its group name for easier distinction
    - `-<TagName>`: retrieve specific tag
    - `-<TagName>=<value>`: write value to tag
    - `-<TagName>=`: delete tag

---

### Archiving

#### 7z (with Encryption)

- Examples:
    - Archive: `7z a -mx=0 -mhe=on -p"<MyPassword>" "<OutputPath>" "<InputPath>"`
    - Archive (password prompt): `7z a -mx=0 -mhe=on -p "<OutputPath>" "<InputPath>"`
    - Extract: `7z x "<PathToArchive>"`
- Flags:
    - `a`: add to archive
    - `x`: extract archive
    - `-mx=<0|9>`: compression level; 0 = none, 9 = max
    - `-mhe=on`: encrypt filenames
    - `-p"<Password>"`: set password without prompting
    - `-p`: prompt for password (safer)

---

#### tar

- Examples:
    - Archive: `tar --create --gzip --file="<OutputPath>.tar.gz" "<InputPath>"`
        - `<InputPath>`: positional; one or more space-separated file/directory paths
    - Extract: `tar --extract --gzip --file="<InputPath>.tar.gz" --directory="<OutputPath>"`
- Flags:
    - `--create[-c]`: create a new archive
    - `--extract[-x]`: extract archive
    - `--gzip[-z]`: applies gzip compression on top of tar
    - `--file[-f]`: specify archive filepath
    - `--directory[-C]`: output directory; defaults to current directory if omitted (extract only)

---

#### gzip

- Examples:
    - Archive: `gzip <file>`
    - Extract: `gzip --decompress <file>.gz`
- Notes:
    - Single files only; no directory support - use `tar.gz` or `7z` instead
    - Replaces original with archive on compress, and replaces archive with original on extract - no copy kept

---

### GPG

- Notes:
    - General usage pattern: `gpg [options] command [args]`

#### GPG Create Strong Key - Interactive

- Examples:
    - Generate key: `gpg --full-generate-key`
- Notes:
    - Interactive prompts, in order:
        - `Please select what kind of key you want:` -> answer `(16) ECC and Kyber`
        - `Please select the Kyber variant you want:` -> answer `(4) Kyber 1024 (X448)`
        - `Please specify how long the key should be valid. (0 = key does not expire)` -> `Key is valid for? (0):` answer `0`
        - `Key does not expire at all, Is this correct? (y/N)` -> answer `y`
        - `Real name:` -> answer `<MyKeyId>`
        - `Email Address:` -> leave blank
        - `Comment:` -> leave blank
        - GUI Prompt: `Enter Passphrase: ***`
        - GUI Prompt: `Confirm Passphrase: ***`
    - Success message:
      ```
      public and secret key created and signed.
      pub   ed448 YYYY-MM-DD [SC]
            <HEX:64>
      uid   <MyKeyId>
      sub   ky1024_cv448 YYYY-MM-DD [E]
            <HEX:64>
      ```

---

#### GPG Create Strong Key - Noninteractive

- Examples:
    - Generate key via heredoc:
      ```shell
      gpg --batch --generate-key <<'EOF'
      %echo Generating ECC (Ed448) + Kyber1024/X448 key
      Key-Type: eddsa
      Key-Curve: ed448
      Key-Usage: sign
      Subkey-Type: kyber
      Subkey-Length: 1024
      Subkey-Curve: cv448
      Subkey-Usage: encrypt
      Name-Real: <MyKeyId>
      Expire-Date: 0
      %commit
      %echo done
      EOF
      ```
- Notes:
    - Equivalent to the `(16) ECC and Kyber` / `(4) Kyber 1024 (X448)` selection in the interactive variant above
    - Passphrase entry is still an interactive GUI popup; nothing else prompts

---
#### GPG Key Management

##### Export Public Key

- Examples:
    - Export: `gpg --armor --output "<MyKeyId>.public.asc" --export "<MyKeyId>"`
- Notes:
    - Verify header: `head -1 "<MyKeyId>.public.asc"` should output `-----BEGIN PGP PUBLIC KEY BLOCK-----`
- Flags:
    - `--armor[-a]`: output ASCII-armored text instead of binary
    - `--output[-o] "<path>"`: write result to file instead of stdout
    - `--export "<KeyId>"`: export public key

---

##### Export Private Key

- Examples:
    - Export: `gpg --armor --output "<MyKeyId>.private.asc" --export-secret-keys "<MyKeyId>"`
- Notes:
    - GUI Prompt: `Enter passphrase: ***`
    - Verify header: `head -1 "<MyKeyId>.private.asc"` should output `-----BEGIN PGP PRIVATE KEY BLOCK-----`
    - Verify encryption: `gpg --list-packets "<MyKeyId>.private.asc"` output should contain `iter+salt S2K`
- Flags:
    - `--armor[-a]`: output ASCII-armored text instead of binary
    - `--output[-o] "<path>"`: write result to file instead of stdout
    - `--export-secret-keys "<KeyId>"`: export private key

---

##### Import Key

- Examples:
    - Import: `gpg --import <MyKeyId>.*.asc`
- Notes:
    - Accepts public and/or private key files; glob pattern above matches both `.public.asc` and `.private.asc`

---

##### List Keys

- Examples:
    - List public keys: `gpg --list-keys`
    - List private keys: `gpg --list-secret-keys`
- Notes:
    - Trust level for each key is shown next to `uid`, e.g. `[ultimate]`, `[full]`, `[unknown]`
- Flags:
    - `--list-keys[-k]`: list public keys
    - `--list-secret-keys[-K]`: list private keys
    - `--fingerprint`: show full fingerprint under each key

---

##### Edit Key (Set Trust)

- Examples:
    - Set trust interactively: `gpg --edit-key "<MyKeyId>" trust`
        - Drops into an interactive prompt asking for a trust level 1-5, where `5 = I trust ultimately`; confirm with `y` if prompted, then it returns to the shell

---

#### GPG Encrypt / Decrypt Files

- Examples:
    - Encrypt file: `gpg --recipient "<MyKeyId>" --output "<OutputPath>.gpg" --encrypt "<InputPath>"`
    - Decrypt file: `gpg --output "<OutputPath>" --decrypt "<InputPath>.gpg"`
- Notes:
    - If `<OutputPath>` already exists, gpg does not overwrite it - it either prompts `Overwrite? (y/N)` (interactive) or fails (non-interactive)
- Flags:
    - `--recipient[-r] "<KeyId>"`: public key to encrypt for
    - `--output[-o] "<path>"`: write result to file instead of stdout
    - `--encrypt[-e] "<InputPath>"`: encrypt input file
    - `--decrypt[-d] "<InputPath>"`: decrypt input file
    - `--trust-model always`: bypass trust checks for this command without modifying the trust database
    - `--batch`: suppress interactive prompts (fail instead of prompting)
    - `--yes`: Assume "yes" on most questions. Should not be used in an option file. 

---


## Linux

### chown / chmod (Permissions)

- Examples:
    - Set owner: `chown <user>:<group> "<path>"`
    - Set mode (octal): `chmod <mode> "<path>"`
    - Set mode (symbolic): `chmod u=rwx,g=rwx,o=rwx "<path>"`
- Notes:
    - Octal mode digits map to owner / group / others, e.g. `755` = `rwxr-xr-x`
    - Symbolic targets: `u` = owner, `g` = group, `o` = others, `a` = all
    - Capital `X` in symbolic mode applies execute only to directories, not regular files

---

### user and group management

#### groupadd / groupdel / members

- Examples:
    - Add group: `groupadd <GroupName>`
    - Delete group: `groupdel <GroupName>`
    - List group members: `members <GroupName>`

---

#### useradd (add user)

- Examples:
    - Add user: `useradd <username>`
- Flags:
    - `--no-create-home[-M]`: do not create a home directory
    - `--create-home[-m]`: create a home directory
    - `--no-user-group[-N]`: do not create a group with the same name as the user
    - `--user-group[-U]`: create a group with the same name as the user
    - `--home-dir[-d] "<path>"`: specify exact home directory path instead of default
    - `--groups[-G] "<group1,group2>"`: make user part of specified groups (CSV)
    - `--system`: create a system account with UID in system interval
    - `--shell <shell>`: specify user shell
        - `--shell /usr/sbin/nologin`: disable interactive login; allows non-interactive commands only
        - `--shell /bin/false`: block all shell access

---

#### usermod (Add User to Group)

- Examples:
    - Add to group: `usermod --append --groups <GroupName> <username>`
- Notes:
    - Always use `--append` when adding to a group - omitting it removes the user from any groups not listed in `--groups`
- Flags:
    - `--append[-a]`: append to existing groups instead of replacing
    - `--groups[-G] "<group1,group2>"`: groups to add; accepts CSV for multiple

---

#### deluser (Remove User from Group)

- Examples:
    - Remove from group: `deluser <user> <group>`

---

### sudo (Run Command as Specific User)

- Examples:
    - Run as user: `sudo --user <username> -- <command>`
- Flags:
    - `--user[-u] <username|UID>`: run as specified user name or ID
    - `--non-interactive[-n]`: non-interactive mode; no prompts
    - `--`: everything after this is the command to execute

---

### at (Schedule Command Execution)

- Examples:
    - Run immediately: `echo "<command>" | at now`
    - Run at time: `echo "<command>" | at <HH:MM>`
    - Run after delay: `echo "<command>" | at now +<Count> <minutes|hours|days|weeks>`
- Notes:
    - The process running the command runs in a non-interactive shell and does not block the current shell

---

### apt (Package Manager)

- Examples:
    - Remove package and config: `apt purge <PackageName>`
    - Remove package, keep config: `apt remove <PackageName>`

---

### Networking

#### wget (Download File from URL)

- Examples:
    - Download with retry: `wget --continue --tries=0 --retry-connrefused --timeout=10 --wait=3 <URL>`
- Flags:
    - `--continue[-c]`: resume a partial download; safe to stop and restart without re-downloading
    - `--tries=<n|0>`: retry attempts; 0 = indefinite
    - `--retry-connrefused`: retry even if server is not currently listening
    - `--timeout=<seconds>`: reconnect if no data received within this duration
    - `--wait=<seconds>`: sleep between reconnect attempts

---

### xargs (Compounding, Run Commands from Piped Input)

- Examples:
    - null-delimited: `find / --name "*.js" -print0 | xargs --replace="{ARG}" --delimiter="\0" echo "found JS file: {ARG}"`
    - newline-delimited: `printf "1\n2 3\n4 5" | xargs --replace="{ARG}" --delimiter="\n" echo "current: {ARG}"`
        - would run commands for `"1"`, `"2 3"`, `"4 5"`
- Notes:
    - Runs a command per token split by the specified delimiter from piped input
    - Example 2 would run commands for `"1"`, `"2 3"`, `"4 5"`
    - Everything after the flags is the freely typed command to run
- Flags:
    - `--replace[-I] "<placeholder>"`: replace default `{}` placeholder with custom token
    - `--delimiter[-d] "<character>"`: character to split piped input on

---

### FileSystem

#### bash (Write File from Inline Content)

- Examples:
    - Heredoc to file: `cat > "<OutputPath>" << 'EOF'`  
      `<FileContent>`  
      `EOF`
- Notes:
    - Quoting the delimiter (`'EOF'`) prevents variable/command expansion inside the block - use unquoted `EOF` to allow substitution
    - `>` overwrites; `>>` appends

---

#### grep (Search String in Files)

- Examples:
    - Basic: `grep --recursive --line-number --ignore-case --fixed-strings "<pattern>" "<dir>"`
    - Regex: `grep --recursive --line-number --ignore-case --perl-regexp "<pattern>" "<dir>"`
    - Filtered by filename: `grep --recursive --line-number --ignore-case --fixed-strings --include="<glob>" "<pattern>" "<dir>"`
- Notes:
    - Last argument is the directory to start scan from
- Flags:
    - `--recursive[-r]`: traverse subdirectories
    - `--line-number[-n]`: print line numbers with matched patterns
    - `--ignore-case[-i]`: case-insensitive match
    - `--fixed-strings[-F]`: literal string match (no regex)
    - `--perl-regexp[-P]`: PCRE regex match, example `^start.*end$`
    - `--include=<glob>`: restrict to matching filenames, glob patterns evaluates filename and not full path, example `*.js`
    - `--exclude=<glob>`: skip matching filenames, glob patterns evaluates filename and not full path, example `*.js`

---

#### shred (Secure delete files)
- Examples:
    - delete file: `shred --remove <FilePath1> <FilePath2>`
- Notes:
    - Not guaranteed effective on journaling, copy-on-write, or SSD/flash
- Flags:
    - `--remove[-u]`: deletes the file after overwriting (without flag the file is overwritten but not deleted or renamed)
    - `--iterations[-n]=<N>`: number of overwrite passes, defaults to 3.
    
---

#### find (Find File or Path by Name)

- Examples:
    - Basic: `find <dir> --name "<filename>*"`
- Notes:
    - First argument is the base search directory
    - `--name` matches the basename (last part of path); use `--path` for full path matching - supports `*` wildcards
    - Flags such as `--type`, `--path`, `--name` can be negated with `!`, e.g. `! --type l ! --type d ! --path "*/cache/*"`
- Flags:
    - `-name "<pattern>"`: match basename of files/folders
    - `-path "<pattern>"`: match full path
    - `-mindepth <n>`: search from this depth level and below (0 includes the start path itself)
    - `-maxdepth <n>`: limit how deep in the folder tree to search
    - `-type <f|d|l>`: filter by type; f = regular files, d = directories, l = symbolic links

---

#### ln (symlink)

- Examples:
    - Soft link: `ln --symbolic "<OriginalFileOrFolderPath>" "<LinkPath>"`
- Notes:
    - Omitting `--symbolic` creates a hard link (files only)
    - Folders in Linux require `--symbolic`; hard links for directories are not supported

---

### Shell Scripting

#### bash (Script Setup)
- Examples:
    - Shebang: `#!/bin/bash`
    - Run script: `bash <scriptName>` or `./<scriptName>` or `bash -c "echo command here..."`
    - Exit immediately on errors, unset vars, and pipeline failures: `set -euo pipefail`
    - Set cwd to running script folder: `cd "$(dirname "$0")"`


---

#### bash (Variable Assignment & Expansion)
- Examples:
    - Assign: `VAR="<value>"`
    - Assign via Command Substitution: `VAR=$(<command>)`
    - Read: `echo "$VAR"` or `echo "${VAR}"`

---

#### bash (Conditional Tests)
- Examples:
    - Path exists: `[[ -e <path> ]]`
    - Is regular file: `[[ -f <path> ]]`
    - Is directory: `[[ -d <path> ]]`
    - Is executable: `[[ -x <path> ]]`
    - String empty: `[[ -z "$VAR" ]]`
    - String not empty: `[[ -n "$VAR" ]]`
    - String equal: `[[ "$a" == "$b" ]]`
    - String not equal: `[[ "$a" != "$b" ]]`
    - Glob match: `[[ "$str" == <pattern> ]]`
        - `<pattern>` must stay unquoted or it's treated as a literal string, not a pattern
        - `?` matches one char, `*` matches zero or more (spans segments, unlike filesystem globs)
        - example: `[[ "$file" == report_v*.txt ]]` matches `report_v2.txt`
    - Regex match: `[[ "$str" =~ <regex> ]]`
        - `<regex>` must stay unquoted or it's treated as a literal string, not a pattern
        - example: `[[ "$phone" =~ ^[0-9]+$ ]]` matches when `$phone` is all digits
    - Numeric equal: `[[ $a -eq $b ]]`
    - Numeric not equal: `[[ $a -ne $b ]]`
    - Numeric less than: `[[ $a -lt $b ]]`
    - Numeric greater than: `[[ $a -gt $b ]]`
    - Numeric less than or equal: `[[ $a -le $b ]]`
    - Numeric greater than or equal: `[[ $a -ge $b ]]`
    - Negate: `[[ ! <condition> ]]`
    - Combine with AND: `[[ <condition1> && <condition2> ]]`
    - Combine with OR: `[[ <condition1> || <condition2> ]]`
- Notes:
    - `[[ ]]` preferred over `[ ]`: avoids word-splitting, supports pattern matching
    - Case-insensitive match: `shopt -s nocasematch` before the `[[ ]]`, `shopt -u nocasematch` after (applies to `==`, glob, and `=~` alike)

---

#### bash (Pause for User Input)
- Examples:
    - Wait for Enter: `read -p "Press Enter to continue..."`

---

#### bash (DateTime)
- Examples:
    - DateTime utc: `datetime=$(date --utc +"%Y-%m-%dT%H:%M:%S%z")` -> "2022-06-16T12:09:37+0000"

---

#### bash (Check Exit Code)
- Examples:
    - Branch on result: `<command>; if [[ $? -eq 0 ]]; then echo "success"; else echo "failed"; fi`
    - Chain on success: `<command1> && <command2>`
    - Chain on failure: `<command1> || <command2>`
- Notes:
    - `$?` holds the exit code of the last executed command; `0` = success

---

#### bash (Run Command in Background)
- Examples:
    - Background job: `<command> &`
    - List background jobs: `jobs`
    - Bring job to foreground: `fg %<jobNumber>`
- Notes:
    - Use `nohup <command> &` to keep the process running after the shell exits

#### bash (Environment Variables)
- Examples:
    - Set for session: `export VAR="<value>"`
    - Unset: `unset VAR`

---

#### bash (If/Else Conditionals)
- Examples:
    - Basic if: `if [[ <condition> ]]; then <command>; fi`
    - If/elif/else: `if [[ <condition1> ]]; then <command1>; elif [[ <condition2> ]]; then <command2>; else <command3>; fi`

---

#### bash (Function Definition)
- Examples:
    - Define and call:
        ```shell
            myFunction() {
                echo "arg1: $1"
            }
            myFunction "<value>"
        ```
- Notes:
    - Positional arguments are accessed as `$1`, `$2`, ... and `$@` for all args
    - `$#` holds the argument count; `$0` holds the script/function name
    - Use `return` for exit codes only (0-255); use `echo` + command substitution to return data

---

#### bash (Loops)
- Examples:
    - Iterate over list: `for <item> in <val1> <val2>; do <command>; done`
    - C-style counter: `for (( i=0; i < <count>; i++ )); do <command>; done`
    - While condition true: `while [[ <condition> ]]; do <command>; done`
    - Until condition true: `until [[ <condition> ]]; do <command>; done`

---

#### bash (Redirection)
- Examples:
    - Redirect stdout, overwrite: `<command> > <file>`
    - Redirect stdout, append: `<command> >> <file>`
    - Redirect stderr: `<command> 2> <file>`
    - Redirect stderr to stdout: `<command> 2>&1`
    - Redirect both streams: `<command> &> <file>`
    - Discard all output: `<command> &> /dev/null`

## Windows

### winget (Package Manager)

- Examples:
    - Search: `winget search <SearchTerm>`
    - Install: `winget install --exact --id <PackageId> --scope user`
- Notes:
    - `https://winstall.app` is a useful UI for browsing and finding package IDs
- Flags:
    - `--exact[-e]`: match package ID exactly; prevents partial-match false positives
    - `--id`: specify the package by its unique ID rather than a fuzzy name search
    - `--scope`: install target: `user` (per-user, no admin needed) or `machine` (system-wide, needs admin); not all packages support both

---

### mklink (symlink)
- Examples:
    - Soft file link: `mklink "<LinkPath>" "<OriginalFilePath>"`
    - Hard file link: `mklink /H "<LinkPath>" "<OriginalFilePath>"`
    - Soft folder link: `mklink /D "<LinkPath>" "<RealFolderPath>"`



---

## Scope

- Spec, not documentation - no prose or storytelling
- Full flag names preferred; short flags listed as reference only
- Alias flags follow directly in square brackets: `--flag[-f]`
- Angle brackets denote user-defined placeholders: `<yourArgumentHere>`
- Pipe-separated angle brackets denote predefined constants: `<admin|normal>`
- Notes and flags are indented as children of their tool block