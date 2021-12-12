---
layout: default
title: Dealing with spaces in Linux file names
--- 

When using the shell, dealing with spaces in file names can be annoying. Here is a one liner command (POSIX compatible!) that can strip all spaces from a file name and replace it with another character. It can also be extended to run in a for loop to work on multiple files in a directory.

### Rename a single file

`mv "file name.txt" $(echo "file name.txt" | tr ' ' '-')`

### Rename all files in a directory

This loops through all files in a directory and uses mv tool to rename them. You must double quote the `$echo piped to tr` command substituion, otherwise you may have have unwanted word splitting and globbing.

`for files in *; do mv "$files" "$(echo $files | tr ' ' '-')"; done`

### Find all files on a system

The Find command may be able to search for all files with spaces on a system, and search deep into a directory tree.