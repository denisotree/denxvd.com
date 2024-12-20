---
title: CLI for Data Professionals
date: 2024-12-19
Description: At first glance, working in the terminal might seem outdated. But mastering the command line unlocks powerful tools that can enhance your efficiency and automate routine tasks. In this article, I’ll share the tools and approaches I personally rely on.
tags:
  - bash
  - linux
  - macos
  - terminal
categories: 
featured_image: /tech/cli-data-analyst.jpg
featured_image_caption: 
summary: At first glance, working in the terminal might seem outdated. But mastering the command line unlocks powerful tools that can enhance your efficiency and automate routine tasks. In this article, I’ll share the tools and approaches I personally rely on.
disableComments: true
draft: false
---
At first glance, working in the terminal (or CLI—command-line interface) might seem outdated. Many professionals prefer graphical interfaces, considering them more user-friendly and intuitive. However, mastering the command line provides access to powerful tools that can significantly boost your productivity and automate repetitive tasks. In this article, I’ll cover the essential tools and approaches I use in my daily work.

P.S. While this article is primarily intended for beginners, experienced analysts might also discover useful cases for tools they already know.

---

## Why Spend Time on This?

![](/tech/data-analyst-cli/photo_2024-12-20_15-45-26.jpg)

Working in the command line can seem challenging, especially if you’re accustomed to graphical interfaces. However, the terminal offers several undeniable advantages:

1. **Speed**: Many tasks can be executed faster in the CLI than through a graphical interface.
2. **Automation**: Scripts and commands enable automation of repetitive processes.
3. **Flexibility**: The CLI provides access to features and configurations often unavailable in GUIs.
4. **Cross-Platform Compatibility**: Most CLI tools work seamlessly on Linux, macOS, and sometimes even Windows.

Ideally, you can develop a workflow where your hands never leave the keyboard. This not only speeds up your work but also improves your focus.

---

## Essential CLI Tools for Data Professionals

### 1. SH | BASH | ZSH and essential utils

In a word—shells. Every technical professional in IT needs to understand how to work with the command line. Knowing how to navigate the directory tree, view file contents, create, delete, and move files and folders are essential skills for any Unix-based operating system (Linux, macOS).

Key commands:
- `cd`: Navigate between directories.
- `ls`: List directory contents.
- `cat`, `less`, `head`, `tail`: View file contents.
- `mkdir`: Create a new directory.
- `rm`: Delete files or directories.
- `mv`: Move or rename files.
- `cp`: Copy files.

Example:
```bash
cd /path/to/directory  
ls -l  
cat file.txt  
```

Additionally, learn about environment variables, stream redirection (`>`, `<`, `|`), and writing basic bash scripts to automate tasks.

---

### 2. `grep`, `sed`, and `awk`

These tools are fundamental for processing and extracting insights from text data.

- **`grep`**: Search for patterns in files.
```bash
# Find all lines containing "error" in log.txt.  
grep "error" log.txt  
```

- **`sed`**: A stream editor for search and replace.
```bash
# Replace "old_text" with "new_text" in a file.  
sed 's/old_text/new_text/g' file.txt > new_file.txt  
```

- **`awk`**: Extract and manipulate column-based data.
```bash
# Output the first and third columns from a CSV file.  
awk -F ',' '{print $1, $3}' data.csv  
```

These tools are indispensable when working with large text files or logs.

---

### 3. `jq`

`jq` is a command-line tool for processing and formatting JSON data. JSON (JavaScript Object Notation) is one of the most popular formats for storing and transmitting data, especially in web development and APIs. `jq` simplifies extracting, filtering, and transforming JSON data from files or streams.

Examples:

- **Pretty-print JSON for readability**:
```bash
cat data.json | jq  
```

- **Extract a specific field**:
```bash
# Extract the "name" field from JSON.  
cat data.json | jq '.name'  
```

- **Filter data**:
```bash
# Extract all array elements where the "age" field is greater than 30.  
cat data.json | jq '.[] | select(.age > 30)'  
```

- **Combine data**:
```bash
# Extract the name and age of each object in an array.  
cat data.json | jq '.[] | {name: .name, age: .age}'  
```

`jq` is an essential tool because the need to format JSON frequently arises, and this is likely the fastest way to handle it.

---

### 4. VisiData

`VisiData` is an incredibly powerful tool for working with tabular data. I’ve written a detailed article about why you should start using it for your current tasks. You can read it [here]({{< relref "visidata" >}}).

`VisiData` combines the power of the command line with the ease of working with tables, making it an indispensable tool for data professionals. It supports a wide range of data formats, including CSV, Excel, JSON, SQLite, and even remote data sources.

#### Why Use VisiData?

1. **Performance**: Load and process large datasets instantly.  
2. **Flexibility**: Filter, sort, and aggregate data directly in the terminal.  
3. **Convenience**: Simple keyboard shortcuts allow you to perform complex operations quickly.  
4. **Cross-Platform Compatibility**: Works seamlessly on all major operating systems.  

#### Key Features:

- **View Data**:  
```bash
# Open the file `data.csv` in an interactive mode.  
vd data.csv  
```

- **Filter Rows**:  
Press `z|` and enter a condition to filter rows (e.g., `age > 30`).

- **Sort Data**:  
Press `[` to sort the current column in ascending order (ASC), and `]` to sort in descending order (DESC).

- **Aggregate Data**:  
Press `Shift+F` to group data by a specific column and calculate the `count` by default. Additional aggregation functions can be accessed by pressing `+`.

- **Export Results**:  
After processing data, save the results to a new file by pressing `Ctrl+s`. You can specify a new file name or overwrite the existing file.

---

### 5. `htop`

`htop` is an enhanced version of the `top` command, which provides information about system processes, CPU usage, memory, and other resources.

Example:
```bash
htop  
```

This tool is invaluable for monitoring how your scripts and notebooks consume system resources in real time and for conveniently "killing" unresponsive processes.

---

### 6. `rsync`

`rsync` is a robust tool for synchronizing files between local and remote systems. It supports incremental data transfer, making it highly efficient.

Example:
```bash
# Synchronize a local directory with a remote server.  
rsync -avz /local/path user@remote:/remote/path  
```

---

### 7. SSH

`ssh` is a tool for securely connecting to remote servers. It’s a fundamental utility for managing remote systems from your terminal.

Example:
```bash
ssh user@remote_host  
```

For transferring files, you can use `scp`:
```bash
scp file.txt user@remote_host:/path/to/destination/  
```

---

### 8. VIM and VIM-like Editors (NeoVim, Helix)

`vim` (Vi IMproved) is one of the most popular text editors for the terminal. Many struggle even to exit it, but learning the basics is well worth it. While simpler editors like `Nano` exist, `vim` provides unparalleled control over text.

Why learn it? Because it allows you to perform tasks directly in the terminal. For example, converting a column of strings:

```bash
val1  
val2  
val3  
```

into a comma-separated list:

```bash
"val1", "val2", "val3",  
```

In `vim`, run the command:
```bash
:%s/.*/"&"/ | %s/\n/, /  
```

- `:%s/.*/"&"/` wraps each line in double quotes.
- `%s/\n/, /` replaces newlines (`\n`) with commas and spaces.

Such tasks arise frequently, and having a quick, efficient way to resolve them is invaluable.

---

### 9. `wget` and `curl`

These are incredibly useful tools, especially if you work with data scraping or downloading. While they share some functionality, they complement each other rather than replace one another.

#### Examples with `curl`:

```bash
# Download a file `file.txt` to the current directory.  
curl -O https://example.com/file.txt  
```

```bash
# Download a file and save it as `my_file.txt`.  
curl -o my_file.txt https://example.com/file.txt  
```

```bash
# Send a POST request with parameters `param1` and `param2`.  
curl -X POST -d "param1=value1&param2=value2" https://api.example.com/endpoint  
```

```bash
# Send JSON data to a server.  
curl -X POST -H "Content-Type: application/json" -d '{"key1":"value1","key2":"value2"}' https://api.example.com/endpoint  
```

```bash
# Add headers like authorization tokens or content types.  
curl -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" https://api.example.com/data  
```

```bash
# Download two files simultaneously.  
curl -O https://example.com/file1.txt -O https://example.com/file2.txt  
```

```bash
# Display only the response headers (e.g., HTTP status, date, content type).  
curl -I https://example.com  
```

```bash
# Follow redirects to the final URL and show all intermediate steps.  
curl -vL https://short.url/link  
```

```bash
# Use basic HTTP authentication to access a protected file.  
curl -u username:password -O https://example.com/protected-file.txt  
```

```bash
# Resume a file download from where it left off.  
curl -C - -O https://example.com/large-file.zip  
```

#### Examples with `wget`:

```bash
# Download a file to the current directory.  
wget https://example.com/file.txt  
```

```bash
# Save a file as `my_file.txt`.  
wget -O my_file.txt https://example.com/file.txt  
```

```bash
# Create a file `urls.txt` with a list of URLs to download:  

https://example.com/file1.txt  
https://example.com/file2.txt  
https://example.com/file3.txt  
```

```bash
# Download all files listed in `urls.txt`.  
wget -i urls.txt  
```

```bash
# Ignore SSL certificate errors (if they are invalid).  
wget --no-check-certificate https://example.com/file.txt  
```

```bash
# Resume a file download from where it left off.  
wget -c https://example.com/large-file.zip  
```

```bash
# Download an entire website for offline viewing.  
wget --mirror --convert-links --adjust-extension --page-requisites --no-parent https://example.com/  
```

- `--mirror`: Enables recursive downloading.  
- `--convert-links`: Adjusts links for offline browsing.

You might argue that all of this can be done in Python, and I’d agree. But for straightforward data pipelines, combining CLI tools is often much faster and more convenient.

---

### 10. Git

I was surprised to find that many professionals aren’t comfortable using `git`—especially in the terminal. Let’s quickly review its purpose and how to use it.

`Git` is a version control system that allows you to track changes in your code, collaborate on projects, and manage file versions. For data professionals, `git` is indispensable, particularly when working on Python scripts, dbt models, Jupyter notebooks, or other data-related projects.

#### Essential `git` Commands:

- **Initialize a repository**:
```bash
# Create a new local repository.  
git init  
```

- **Clone a remote repository**:
```bash
# Copy a remote repository to your computer.  
git clone https://github.com/user/repo.git  
```

- **View repository status**:
```bash
# Show the current changes in files (added, modified, or deleted).  
git status  
```

- **Stage changes**:
```bash
# Add a file to the staging area.  
git add file_name  
```

```bash
# Add all files.  
git add .  
```

- **Commit changes**:
```bash
# Save the changes with a description.  
git commit -m "Description of changes"  
```

- **View commit history**:
```bash
# Display a list of all commits in the repository.  
git log  
```

- **Push changes to a remote repository**:
```bash
# Upload changes from the local repository to a remote one (e.g., on GitHub).  
git push origin branch_name  
```

- **Pull changes from a remote repository**:
```bash
# Synchronize the local repository with the remote one.  
git pull origin branch_name  
```

- **Create a new branch**:
```bash
# Create a new branch named "branch_name".  
git branch branch_name  
```

- **Switch between branches**:
```bash
# Switch to the branch "branch_name".  
git checkout branch_name  
```

- **Merge branches**:
```bash
# Merge changes from "branch_name" into "master".  
git checkout master  
git merge branch_name  
```

```bash
# Merge changes from "master" into "branch_name" (e.g., for testing or pull requests).  
git checkout branch_name  
git merge master  
```

#### Why `git` Matters for Data Professionals:

1. **Collaboration**: Data projects often involve multiple contributors. `Git` helps manage changes efficiently and avoid conflicts.
2. **Version History**: You can always revert to an earlier version of your code or data.
3. **Experimentation**: Use branches to test new hypotheses or approaches without affecting the main project version.
4. **Integration with Platforms**: Platforms like GitHub or GitLab make it easy to share projects, conduct code reviews, and automate workflows with CI/CD.

Mastering `git` isn’t just about code management—it’s about organizing your projects effectively. It’s a tool that boosts productivity and confidence when working on data projects.

---

## Additional CLI Tips

1. **Use Autocompletion**: Many shells (like Bash or Zsh) support autocompletion for commands and arguments.
2. **Create Aliases**: Simplify frequently used commands by adding aliases to your `.bashrc` or `.zshrc`.

```bash
alias ll="ls -la"  
alias gs="git status"  
```

3. **Learn Regular Expressions**: They’ll help you work with text data more efficiently.
4. **Document Your Commands**: Write scripts for repetitive tasks.

---

## Postscript

This is by no means a comprehensive list of useful command-line tools. There are countless others, and new ones are created every day. I’ve highlighted the tools that can significantly enhance a data professional’s daily workflow. Mastering these tools is an investment in your productivity and professionalism. Even if you’re just starting your data journey, knowing CLI tools gives you a competitive edge and helps you solve tasks faster and more effectively.

Don’t hesitate to experiment and explore new utilities and approaches. The terminal is a powerful ally that can become your best tool for working with data. Good luck!
