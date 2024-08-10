# Terminal Tips

## Intro

This is a collection of useful terminal commands and tips that I have found helpful in my work as a software engineer.
These tips may cover a variety of topics, including file management, text processing, and system administration in the
future. I hope you find them useful in your own work.

I'm reaaaally tired of having to google the same commands over and over again when I need them, so I decided to create
this repository to keep track of them and share them with others. Instead of searching through multiple websites and
forums, you can find the commands you need in one place and perhaps ask some AI models to modify them to suit your
needs.

I will be adding more tips and commands to this repository over time, so be sure to check back for updates.

## Table of Contents

- [Terminal Tips](#terminal-tips)
    - [Intro](#intro)
    - [Table of Contents](#table-of-contents)
    - [File Management and Searching](#file-management-and-searching)
        - [Sorting Files and Folders by Size](#sorting-files-and-folders-by-size)
        - [Find and Search Files with Specific Criteria](#find-and-search-files-with-specific-criteria)
        - [Comparing Files](#comparing-files)
        - [View Last 100 Lines of a Log File](#view-last-100-lines-of-a-log-file)
        - [Display the 10 Largest Running Processes by Memory Usage](#display-the-10-largest-running-processes-by-memory-usage)
        - [Find and Delete Empty Files and Directories](#find-and-delete-empty-files-and-directories)
        - [Batch Rename Files](#batch-rename-files)
    - [Efficiency](#efficiency)
    - [Contributing](#contributing) </br>
A collection of useful terminal commands and tips.

## File Management and Searching

### Sorting Files and Folders by Size

To sort files and folders by size within a directory, including nested folders, use the following command: (don't forget
to cd into the directory you want to sort)

```bash
du -ah . | sort -hr | head -n 10
```

This command is useful for identifying large files and directories that may be taking up significant disk space,
allowing you to manage your storage more effectively. Let's break down the commands and options used in this example:

- `du -ah .` lists all files and folders in the current directory, including hidden files and folders, with their
  respective sizes. The -h flag makes the output human-readable, and the -a flag includes all files and folders, not
  just directories.

- `sort -hr` sorts the output in reverse order (largest to smallest) based on the human-readable file sizes. The -h flag
  tells sort to consider the human-readable sizes, and the -r flag reverses the order.

- `head -n 10` displays only the top 10 lines of the output, which correspond to the largest files and folders in the
  directory. You can adjust the number after the -n flag to display a different number of items.

### Find and Search Files with Specific Criteria

To find and search for files with specific criteria, such as a certain text within the files, you can use a combination
of find, xargs, and grep commands. For example, to search for files containing the text "invoice" within a directory and
its subdirectories, you can use the following command:

#### Old-School Way: find+grep

```bash
LANG=C find . -maxdepth 4 -type f -size -5M -name "*.pdf" -print0 | xargs -0 -P 8 -I {} grep -il "invoice\|receipt" {}
```

This command combines `find`, `xargs`, and `grep` to search for files containing a specific text ("invoice" in this
case)
within a directory and its subdirectories, while applying several criteria. Let's break down the commands and options
used in this example:

*`LANG=C`*:

- Sets the locale to C, which can make the search faster by using the default C locale for character processing.
- note: this is not necessary for all systems, but it can help speed up the search in some cases.

*`find . -maxdepth 4 -type f -size -5M -name "*.pdf" -print0`*:

- `.`: Starts searching from the current directory.
- `-maxdepth 4`: Limits the search to 4 levels of subdirectories.
- `-type f`: Searches only for files (not directories).
- `-size -5M`: Finds files smaller than 5 MB.
- `-name "*.pdf"`: Searches for files with the .pdf extension. You can replace "*.pdf" with other file extensions or
  patterns or remove this option to search all files.
- `-print0`: Outputs the filenames followed by a null character (\0), which is useful for handling filenames with
  special characters or spaces. This is used with xargs to process the filenames.

*`xargs -0 -P 8 -I {} grep -il "invoice\|receipt" {}`*:

- `-0`: Tells xargs to expect null-terminated strings (used with -print0 from find).
- `-P 8`: Runs up to 8 processes in parallel to speed up the search.
- `-I {}`: Replaces {} with the file names passed by find.
- `grep -il "invoice" {}`: Searches within each file for the term "invoice".
    - `i`: Ignores case sensitivity.
    - `l`: Prints only the names of files that contain the matched text.
    - `"invoice\|receipt"`: Searches for either "invoice" or "receipt" in the files. You can modify this pattern to
      match
      different criteria.

#### Modern Way: ripgrep-all (rga)

Alternatively, you can use the `ripgrep-all` (rga) tool written in rust, which is a faster and more feature-rich alternative to the
traditional grep command. To search for files containing the text "invoice" within a directory and its subdirectories
using rga, you can use the following command:

**Installation**:
To use this command, you'll need to have ripgrep-all installed. You can find the installation instructions on the
[ripgrep-all](https://github.com/phiresky/ripgrep-all) GitHub page.

```bash
rga -il -e 'invoice' -e 'receipt' --max-depth=1 --max-filesize=1M --type=pdf

```

This command uses ripgrep-all (rga), an enhanced version of ripgrep, to search for text in various file types, and then
copies matching PDF files to a target directory.

- `rga`: Runs ripgrep-all, a tool that extends ripgrep to search within PDFs and other file types.
- `-i`: Ignores case sensitivity.
- `-l`: Lists only the names of files containing the matched text.
- `-e 'invoice' -e 'receipt'`: Searches for either "invoice" or "receipt" within the files.
- `--max-depth=1`: Limits the search to the current directory (no deeper subdirectories).
- `--max-filesize=1M`: Limits the search to files smaller than 1 MB.
- `--type pdf`: Restricts the search to PDF files.

#### Copy Found Files to a New Directory

**via `cp` command**:

How to copy the files found by the search command to a new directory? Well, you can use the `xargs` command to execute
the `cp` command for each file found. Here's how you can do it:

```bash
rga -il -e 'invoice' -e 'receipt' --max-depth=1 --max-filesize=1M --type=pdf | xargs -I {} cp {} ./invoice/

```

This command copies each file found by the search command to the `./invoice/` directory using the `cp` command.

- Note: If the target directory does not exist, you can create it using the `mkdir` command.

**via `rsync` command**:

The `cp` command is cool, but I prefer to use the `rsync` command for copying files because it provides more options and
control over the copying process. Here's how you can use `rsync` to copy files found by the search command to a new
directory:

```bash
rga -il -e 'invoice' -e 'receipt' --max-depth=1 --max-filesize=1M --type=pdf | xargs -I {} rsync -a --ignore-existing {} ./invoice/

```

This command copies each file found by the search command to the
`./invoice/` directory using rsync.

- `-a`: Preserves the file attributes and permissions during the copy.
- `--ignore-existing`: Skips copying files that already exist in the target directory.
- You can modify the rsync options to suit your specific requirements. For example, you can use `--remove-source-files`
  to delete the source files after copying them.


### Comparing Files

To compare the contents of two files side by side and highlight their differences, use the following command:
```bash
diff -y file1.txt file2.txt
```
This command shows the differences between file1.txt and file2.txt side by side.

- `diff`: The command used to compare files line by line.
- `-y`: The option that displays the files side by side for easier comparison.

This command is particularly useful for reviewing changes between two versions of a file or comparing the output of two similar files.

### View Last 100 Lines of a Log File

To view the most recent entries in a log file, you can use:

```bash
tail -n 100 /var/log/syslog
```
 - `tail`: The command used to output the end of files.
- `-n 100`: Specifies that you want to view the last 100 lines of the file.
- `/var/log/syslog`: Path to the log file. Replace this with the path to your specific log file!
 
This command is useful for quickly reviewing the latest entries in a log file, which can help in debugging or monitoring system activity.

### Display the 10 Largest Running Processes by Memory Usage

To find the top 10 processes consuming the most **memory** on your system, use:

```bash
ps aux --sort=-%mem | head -n 10
```
 
- `ps`: Command used to display information about running processes.
- `aux`: Shows detailed information about all running processes.
- `--sort=-%mem`: Sorts the processes by memory usage in descending order.
- `head`: Command used to display the beginning of files.
- `-n 10`: Specifies that you want to see only the top 10 entries.

These commands definitely help identify which processes are using the most memory, which can be useful for performance tuning! Well, they do wonders for me.

### Find and Delete Empty Files and Directories

To locate and remove empty files and directories within a specified directory, use:
```bash 
find /path/to/directory -empty -delete
```

- `find`: Command used to search for files and directories.
- `/path/to/directory`: Path where the search should begin.
- `-empty`: Matches empty files and directories.
- `-delete`: Deletes the matched files and directories.

This command is useful for cleaning up your filesystem by removing unnecessary empty files and directories.

### Batch Rename Files
To rename multiple files at once, such as changing all .jpg files to a new format, use:

```bash
mmv "*.jpg" "image_#1.jpg"
```

- `mmv`: Command used for batch renaming files.
- `"*.jpg"`: Pattern to match all .jpg files.
- `"image_#1.jpg"`: New name format where #1 is a placeholder for a sequential number.


To batch rename files, such as renaming all `.jpg` files to `image_1.jpg`, `image_2.jpg`, etc., you can use it.

## Efficiency

To streamline your workflow, you can create aliases for commonly used commands. Add these aliases to your .bashrc or .zshrc file:

```bash
alias ll='ls -lh'
```
This creates a shortcut ll to list files and directories in long format with human-readable sizes.
```bash
alias gs='git status'
```
 This creates a shortcut gs to check the status of your Git repository :)

## Contributing

If you have a useful terminal tip or command that you would like to share, please feel free to contribute by creating a
pull request. I welcome any contributions that can help expand this collection and make it more valuable to others.