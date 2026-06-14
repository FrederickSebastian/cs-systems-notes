
### Chapter 14 Package Management

1. Finding a package in a repository:
	- Debian: `apt-get update; apt-cache search`*`search_string`* 
	- Red hat: `yum search`*`search_string`* 
2. Installing a package from a repository:
	- Debian: `apt-get update; apt-cache install`*`search_string`* 
	- Red hat: `yum install` *`search_string`* 
3. Install a package from a package file:
	- Debian: `dpkg -i` *`package_file`* 
	- Red hat: `rpm -i` *`package_file`* 
4. Removing a package:
	- Debian: `apt-get remove` *`package_name`* 
	- Red hat: `yum erase package_name` *`package_name`* 
5. Update package from the repository:
	- Debian: `apt_get updage; apt-get upgrade`
	- Red hat: `yum update`
6. Update package from the file:
	- Debian: `dpkg -i` *`package_file`*
	- Red hat: `rpm -U` *`package_file`*
7. Listing Installed packages:
	- `dpkg -l`
	- `rpm -qa`
8. Show the information of the installed package:
	- `apt-cache show` *`package_name`*
	- `yum info` *`package_name`*
9. We can also determine whether a package is installed by the parameter `-l` in Debian and `-qa` in Red hat.

### Chapter 15 Storage media

1. `mount`: mount file systems
2. `unmount` 
3. `fdisk` : manipulating partitions

### Chapter 16 Network

1. `ping`
2. `ip`
3. `netstat`: print network statistics
4. `ftp` : internet file transfer program
5. `ssh`: *openSSH* SSH client remote login


### Chapter 17 Search files

1. `find`: find files or directories for certain criteria. Here are the types: 
	- File types
	- Size
	- Name
	- Logical relationships(operators)
	- Predefined actions
	- and so on

2. `xargs` : accepts input from standard input and converts it into an argument list for a specified command, for example: 
	`find ~ -type f -name 'foo*' -print | xargs ls -l` 

### Chapter 18 Archiving and Backup

1. `gzip` : Lossless compressing, **replace the original file with compressed version**. If the object is a directory, need to recursive(-r).
2. `gunzip` : restore compressed files
3. `bzip2` and `bunzip2` are same to `gzip` except the option recursive(-r)
4. **Caution: do not try to compress a file that has already been compressed such .jpg, .mp3 file.**

### Chapter 19  Regular Expressions

1. `grep` : global regular expression print. The syntax:
	`grep [options] regex [file...]`
	- the *regex* is a regular expression, which is the object string.

