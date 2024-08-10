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
    - [Contributing](#contributing)

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

## Contributing

If you have a useful terminal tip or command that you would like to share, please feel free to contribute by creating a
pull request. I welcome any contributions that can help expand this collection and make it more valuable to others.