# 05. Redirection & Process Management (Ch-10, Ch-11)

## 1. I/O Streams & Redirection (Ch-10)

| Descriptor | Name | Default |
| :--- | :--- | :--- |
| `0` | stdin | Keyboard |
| `1` | stdout | Terminal |
| `2` | stderr | Terminal |

```bash
ls > ls_output.txt           # Create/overwrite file with stdout
ls >> ls_output.txt          # Append stdout to file
find /etc . -name hostname 2> error.txt   # Redirect stderr only
find /etc -name hostname 2> /dev/null     # Discard errors

# Redirect Both stdout and stderr
find /etc -name hostname &> find_output.txt          # Bash shorthand
find /etc -name hostname > find_output.txt 2>&1      # Portable form (order matters!)
find /etc -name hostname >> find_output.txt 2>&1     # Append both

# Input Redirection & Pipelines
tr 'a-z' 'A-Z' < /etc/hosts  # Read stdin from file
ls -l | tee homedir.txt      # Write stdout to file AND display it
ls -l | tee -a homedir.txt   # Append to file AND display it
echo 'x1 x2 x3 x4' | xargs touch   # Build and run touch from stdin items
find . -name "*.tmp" | xargs rm    # Delete files found by find

# Here-document / here-string
cat << EOF > notes.txt
line one
line two
EOF
grep root <<< "root:x:0:0"
```

---

## 2. Process Monitoring & Priorities (Ch-11)

```bash
# Inspection
ps -ef                       # Full listing of all processes
ps aux                       # BSD-style listing with CPU/MEM usage
ps -u bob                    # Processes of one user
pstree -p                    # Process tree with PIDs
pgrep sshd                   # Find PIDs by name
uptime                       # Uptime and load averages
free -h                      # Memory statistics

# Real-time Monitoring
top                          # Live process view (q quits; k kills; r renices)
htop                         # Friendlier top (if installed)
watch -n 15 tail /var/log/ndg/web.log   # Re-run every 15s (-d highlights changes)

# Signals & Termination
kill 1234                    # Send SIGTERM (15) to PID 1234
kill -9 1234                 # SIGKILL: force kill, cannot be caught
kill -HUP 1234               # SIGHUP: often tells daemons to reload config
kill -l                      # List all signal names
killall sleep                # Terminate all processes named 'sleep'
pkill -u bob                 # Kill processes by user or pattern

# Process Niceness (-20 = highest priority, 19 = lowest)
nice -n 10 sleep 100 &       # Start with lowered priority
renice -n 15 -p 138          # Change priority of running PID 138
ps -o pid,ni,comm -p 138     # View a process's nice value
```

---

## 3. Shell Jobs & Terminal Multiplexers (Ch-11)

```bash
# Shell Job Control
sleep 100 &                  # Launch job in background
jobs                         # List shell jobs
# Press Ctrl+Z               # Suspend the foreground process
bg %1                        # Resume job 1 in the background
fg %1                        # Bring job 1 to the foreground
nohup ./script.sh &          # Keep running after logout
disown %1                    # Detach job from the shell

# Screen Sessions
screen                       # Start a screen session
# Press Ctrl+A, then d       # Detach from session
screen -ls                   # List sessions
screen -r                    # Reattach to session

# Tmux Multiplexer
tmux new-session 'top'       # Start tmux running 'top'
tmux new -s work             # Start a named session
# Press Ctrl+B, then d       # Detach from tmux session
tmux ls                      # List sessions
tmux attach -t work          # Reattach to a named session
```

### ➕ Additional Essentials: Open Files & Resources

```bash
lsof -i :80                  # Which process uses port 80
lsof -u bob                  # Files opened by a user
vmstat 2 5                   # Memory/CPU/IO stats, 5 samples 2s apart
iostat -x 2                  # Disk I/O stats (package: sysstat)
```
