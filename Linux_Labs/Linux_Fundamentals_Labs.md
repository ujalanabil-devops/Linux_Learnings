### Create the lab directory:
```
mkdir -p ~/linux-labs
cd ~/linux-labs
```
### Lab 1 — Filesystem Exploration
- Objective:Understand the Linux filesystem hierarchy.
```
Run:
pwd
ls -lah /

Identify:
/etc
/var
/home
/tmp
/opt
/usr
/dev
/proc
/sys
/run

Inspect:
ls -lah /etc | head
ls -lah /var/log | head
ls -lah /home
ls -lah /tmp
```
### Lab 2 — pwd, cd, ls
- Create the lab structure:
```
cd ~/linux-labs
mkdir -p project/{config,logs,data,backup,scripts}

Navigate:
cd project
pwd

cd config
pwd

cd ..
pwd

cd -
pwd

Practice:
ls
ls -l
ls -la
ls -lah

Challenge:- 
Without using cd more than twice, get from:
~/linux-labs/project/config

to:

~/linux-labs/project/logs
```
### Lab 3 — cp, mv, rm
```
Create files:
cd ~/linux-labs/project

echo "production configuration" > config/app.conf
echo "application started" > logs/app.log

Copy:
cp config/app.conf config/app.conf.bak

Verify:
ls -lah config/

Rename:
mv config/app.conf.bak config/app.conf.backup

Move:
mv config/app.conf.backup backup/

Delete safely:
rm -i backup/app.conf.backup

Challenge
Create:

data/
├── app1.txt
├── app2.txt
└── app3.txt

Then copy all three files to backup/.
```
### Lab 4 — cat, less, head, tail
```
Create a realistic log:
cd ~/linux-labs/project

for i in {1..100}; do
    echo "$(date '+%Y-%m-%d %H:%M:%S') INFO Request processed ID=$i" >> logs/app.log
done

Add errors:
echo "ERROR Database connection failed" >> logs/app.log
echo "WARNING High memory usage" >> logs/app.log
echo "ERROR API request failed" >> logs/app.log

Practice:
cat logs/app.log
For large files, use:
less logs/app.log

Then:
head logs/app.log
head -n 20 logs/app.log
tail logs/app.log
tail -n 20 logs/app.log

Live log practice
Terminal 1:
tail -F logs/app.log

Terminal 2:
echo "ERROR New production error" >> ~/linux-labs/project/logs/app.log

Watch the first terminal.
This simulates real-time production log monitoring.
```
### Lab 5 — grep
```
Search:
grep "ERROR" logs/app.log

Case insensitive:
grep -i "error" logs/app.log

Line numbers:
grep -n "ERROR" logs/app.log

Count:
grep -c "ERROR" logs/app.log

Search multiple patterns:
grep -Ei "error|warning" logs/app.log

Invert:
grep -v "INFO" logs/app.log

Production challenge
Find:
ERROR
WARNING
FAILED
CRITICAL
with one command.
```
### Lab 6 — Pipes |
```
Practice:
cat logs/app.log | grep ERROR

Then:
grep ERROR logs/app.log | tail -5

Then:
grep ERROR logs/app.log | wc -l

Try:
ls -lah /var/log | grep log

Challenge
Build a command that:
Finds errors and displays only the last 5.

Expected approach:
grep -i "error" logs/app.log | tail -5
```
### Lab 7 — Redirection
```
Create:
echo "Linux DevOps Lab" > test.txt

Check:
cat test.txt

Append:
echo "Day 1" >> test.txt
echo "Day 2" >> test.txt

Check:
cat test.txt

Standard error
Try:
ls /does-not-exist

Redirect the error:
ls /does-not-exist 2> errors.txt

Check:
cat errors.txt

Discard errors:
ls /does-not-exist 2>/dev/null
Challenge
Create:
output.txt
error.txt
where normal output goes to output.txt and errors go to error.txt.
```
### Lab 8 — find
```
Create test files:
cd ~/linux-labs/project
touch data/app.log
touch data/database.log
touch data/test.txt
touch data/config.conf

Find .log files:
find data -name "*.log"

Find .conf:
find data -name "*.conf"

Find files:
find data -type f

Find directories:
find data -type d

Find files modified within the last day:
find data -type f -mtime -1

Production challenge
Find all .log files under your lab directory:
find ~/linux-labs -type f -name "*.log"
```
### Lab 9 — locate
```
Install if necessary:
sudo apt update
sudo apt install plocate

Update database:
sudo updatedb

Search:
locate ssh_config

Compare:
find /etc -name "ssh_config"
Understand the difference
locate uses a database.
find searches the filesystem in real time.
locate is generally faster.
find is more precise and supports more filtering.
```
### Lab 10 — sort and uniq
```
Create data:
cat > users.txt <<EOF
alice
bob
alice
john
bob
alice
john
john
john
EOF

Sort:
sort users.txt

Remove duplicates:
sort users.txt | uniq

Count occurrences:
sort users.txt | uniq -c

Sort by frequency:
sort users.txt | uniq -c | sort -nr

Production challenge
Find the most common error messages in your log:
grep ERROR logs/app.log | sort | uniq -c | sort -nr
```
### Lab 11 — cut
```
Practice with /etc/passwd:
cut -d: -f1 /etc/passwd

Get username and shell:
cut -d: -f1,7 /etc/passwd

Create your own data:
cat > employees.txt <<EOF
101:Alice:DevOps
102:Bob:Linux
103:John:Cloud
104:Sarah:SRE
EOF

Extract names:
cut -d: -f2 employees.txt

Extract departments:
cut -d: -f3 employees.txt
```
### Lab 12 — xargs
```
Create:
cat > files.txt <<EOF
file1.txt
file2.txt
file3.txt
EOF

Practice safely:
cat files.txt | xargs -n1 echo

Create files:
touch file1.txt file2.txt file3.txt

Then:
cat files.txt | xargs -n1 ls -lh

Important:-
Before using xargs with destructive commands such as rm, test first with:
xargs -n1 echo
```
### Lab 13 — Wildcards
```
Create:
touch app.log
touch database.log
touch server.log
touch test.txt
touch app.conf

Find: ls *.log

Then:
ls *.conf

Try:
ls app*

Try:
ls ?.log

Try:
ls [asd]*.log

Understand what each wildcard matches.
```
### Lab 14 — Quoting
```
Run:
name="Production Server"

echo $name
echo "$name"
echo '$name'

Now:
file="my application.log"
touch "$file"
ls -l "$file"

Compare:
ls $file

with:

ls "$file"

Production lesson:-
Always be careful with variables:

"$file"
"$directory"
"$username"

Quoting helps protect filenames and variables containing spaces or special characters.
```
### Lab 15 — Environment Variables
```
Check: env

Check PATH: echo "$PATH"

Create: export APP_ENV=production

Check: echo "$APP_ENV"

Create more:
export APP_NAME=myapp
export APP_PORT=8080

Check: env | grep APP

Remove: unset APP_PORT

Verify:
echo "$APP_PORT"
Lab 16 — PATH Troubleshooting

Create a script:
mkdir -p ~/linux-labs/bin

cat > ~/linux-labs/bin/hello-devops <<'EOF'
#!/bin/bash
echo "Hello DevOps"
EOF

Make it executable:
chmod +x ~/linux-labs/bin/hello-devops

Try:
hello-devops

You should get:
command not found

Why?
Because the directory is not in $PATH.

Check:
echo "$PATH"

Temporarily add it:
export PATH="$HOME/linux-labs/bin:$PATH"

Now:
hello-devops

Check:
command -v hello-devops

This is a useful real-world DevOps troubleshooting exercise.
```
### Lab 17 — Bash History
```
Run:
pwd
ls
whoami
date
uptime

Check:
history

Use interactive search:
Ctrl + R

Search for:
uptime

Run previous command:
!!

Check recent history:
history | tail

Production lesson
Never put passwords or API keys directly into commands because they may appear in shell history.
```
### Lab 18 — Aliases
```
Create:
alias ll='ls -lah'

Test:
ll

View:
alias

Remove:
unalias ll

Create another:
alias logs='cd ~/linux-labs/project/logs'

Test:
logs
pwd

For permanent aliases:
nano ~/.bashrc

Add:
alias ll='ls -lah'

Reload:
source ~/.bashrc
```
### Lab 19 — Production Log Investigation ⭐

Now combine everything.
```
Create a realistic log:

cd ~/linux-labs/project

for i in {1..500}; do
    echo "$(date '+%Y-%m-%d %H:%M:%S') INFO Request processed ID=$i" >> logs/production.log
done

for i in {1..10}; do
    echo "$(date '+%Y-%m-%d %H:%M:%S') ERROR Database connection failed" >> logs/production.log
done

for i in {1..5}; do
    echo "$(date '+%Y-%m-%d %H:%M:%S') WARNING High memory usage" >> logs/production.log
done

Question 1 — How many errors?
grep -c ERROR logs/production.log
Question 2 — Show the last 5 errors
grep ERROR logs/production.log | tail -5
Question 3 — Count error types
grep ERROR logs/production.log | sort | uniq -c
Question 4 — Show latest important messages
grep -Ei "ERROR|WARNING|CRITICAL|FAILED" logs/production.log | tail -10
```
### Lab 20 — Production Disk Investigation ⭐⭐⭐
```
Create large test files safely inside your lab:

cd ~/linux-labs/project

dd if=/dev/zero of=data/test1.img bs=1M count=100
dd if=/dev/zero of=data/test2.img bs=1M count=200
dd if=/dev/zero of=data/test3.img bs=1M count=300

Check:
du -sh data/*

Sort:
du -sh data/* | sort -h

Find files over 100 MB:
find data -type f -size +100M -exec ls -lh {} \;

Production mindset
If /var were full, the investigation would look like:

df -h
   ↓
Identify full filesystem
   ↓
du
   ↓
Identify large directory
   ↓
find
   ↓
Identify large files
   ↓
Check logs
   ↓
Determine root cause
```
### Lab 21 — Complete Production Simulation ⭐⭐⭐
```
Scenario

You receive this alert:

Application server: Disk usage above 85%. Users are reporting errors.

You have no instructions other than the alert.

Your job is to investigate.

Start with:

df -h

Then determine:
Which filesystem is affected?
Which directory is consuming space?
Which files are largest?
Are logs responsible?
Are there old files?
Are there errors in the application log?
Is the application still running?

Useful commands:
df -h
du -xhd1
find
ls -lh
grep
tail
head
sort
uniq
ps
Important

Don't immediately delete anything.

First identify the root cause.

This is one of the most important habits in production troubleshooting.
```
### Final Challenge — DevOps Linux Assessment
```
Try these without looking at your notes.

Task 1

Find all .log files under /var/log.

Task 2

Find all files larger than 100 MB under your home directory.

Task 3

Find the top 10 largest files in your lab.

Task 4

Count the number of ERROR entries in your production log.

Task 5

Display the latest 20 errors.

Task 6

Find the most common error message.

Task 7

Extract all usernames from /etc/passwd.

Task 8

Find all commands containing ssh in your shell history.

Task 9

Create a command that displays the top 10 largest directories in your lab.

Task 10

Create a script directory that is not in $PATH, then troubleshoot why the command cannot be found.
```
#### Production troubleshooting
```
DevOps Learning Goal
Don't just memorize commands.
You should be able to receive a production alert such as:
"The Linux server is running out of disk space."

and immediately think:

df -h
   ↓
Identify filesystem
   ↓
du -xhd1
   ↓
Identify large directory
   ↓
find
   ↓
Identify large files
   ↓
Inspect logs
   ↓
Determine root cause
   ↓
Clean/rotate safely
   ↓
Verify with df -h

This troubleshooting mindset is more valuable for a DevOps engineer than memorizing hundreds of Linux commands.
```