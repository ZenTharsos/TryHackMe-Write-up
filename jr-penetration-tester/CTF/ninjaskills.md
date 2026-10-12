# TryHackMe - Ninja Skills
<img width="1051" height="585" alt="ninjaskills_room" src="https://github.com/user-attachments/assets/bbb4e058-8ee0-41b9-9c8e-bb10846c921f" />

### What Is This Challenge About:

The Ninja Skills room help learners practise Linux command-line skills.
The task is to locate specific files and examine their contents
to answer the question provided.

### What Does This Challenge Teach:
- How to used Linux commands to search for files.
- How to locate files by names and other criteria.
- How to examine file contents.
- How to combine commands to complete task more efficiently.

### My Approach 

I used Linux command-line tools to search for the requested
files and investigate their contents.

**First i will connet to the lab machine with provided ssh cridential**
- username: new-user
- password: new-user

<img width="717" height="359" alt="image" src="https://github.com/user-attachments/assets/fde00032-7015-44fb-8cee-20c36cf43e5e" />


##Task 1: Which of the above files are owned by the best-group group


The first Question is to find files that are owned by the **best-group** group.

<img width="684" height="71" alt="Screenshot_2026-10-13_11-31-27" src="https://github.com/user-attachments/assets/a6b8707d-0d68-4f6a-a03b-1d2a783de8dd" />

The command is quite straight forward
```
find / -type f -group best-group -exec ls -la {} + 2>/dev/null
```
**What the commands does:**
- find / : Start searching from the root directory.
- -type f : search for only file and nothing else.
- -group : search for a specific group.
- -exec : take the command after and run it.
- ls -la : list file/directories in a long format, include hidden file/directories.
- 2>/dev/null : send all the error to a black hole.


##Task 2: Which of these files contain an IP address?

[new-user@ip-10-64-171-87 ~]$ find / -type f \( -iname 8v2l -o -iname bny0 -o -iname c4zx -o -iname d8b3 -o -iname fhl1 -o -iname oimo -o -iname pfbd -o -iname rmfx -o -iname srsq  -o -iname uqyw -o -iname v2vb -o -iname x1uy \) -exec  grep -o -E '([0-9]{1,3}\.){3}[0-9]{1,3}' *  {} \; 2>/dev/null


<img width="857" height="191" alt="ip_20-10-13_12-14-04" src="https://github.com/user-attachments/assets/da313b09-1c13-4d2f-acf6-5afb4eed58da" />


**What The Command does:**

- The command start the search from the root directory for file provided
- -iname : simply make the search cases insensitive.
- -o : simply means "OR".
- grep : search for patterns in a file .
- -o : this simply tell grep to show a matching part in the line.
- E : this is and extended regular expression.



##Task 3: Which file has the SHA1 hash of 9d54da7584015647ba052173b84d45e8007eba94



