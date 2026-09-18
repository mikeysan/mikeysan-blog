---
title: "Manage Download Folder using Python"
author: "Mikey San"
date: 2023-03-28T08:50:09.206Z
lastmod: 2026-09-18T13:13:38+01:00

description: ""
subtitle: ""

image: "1.jpg" 
images:
 - "1.jpg"

aliases:
  - "/manage-download-folder-using-python-1779be27a40f"

---


![](1.jpg)

Photo by [Gabriel Heinzer](https://unsplash.com/@6heinz3r?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

I was following a tutorial today about automating things using Pythons. One of the scripts covered was how to move files older than 30 days in the download folder to another folder, so in this article, we will go over how to write a script that scans your download folder and move files to another folder called “to\_delete”.

**Step 1: Import Modules**

To begin, we need to import the required modules. In this case, we will be using the os and shutil modules. The os module provides a way to interact with the file system, and the shutil module provides a way to move files from one location to another.

```python
import os
import shutil
```

**Step 2: Define the Variables**

Next, we need to define the variables that we will be using in our script. Here we need two variables. One to hold the path to our download folder, and the other to hold the path to where we will move the files to.

```python
download_folder = "[path to download folder]"
to_delete_folder = "[path to to_delete folder]"
```

**Step 3: Scan the Download Folder**

Now, we need to scan the download folder and identify the files that are older than 30 days. We can do this by using the os.path.getmtime() function, which returns the last modification time of a file in seconds since the epoch.

```python
for file in os.listdir(download_folder):
    path = os.path.join(download_folder, file)
    if os.path.isfile(path):
        if os.path.getmtime(path) < (time.time() - 30 * 86400):
            shutil.move(path, to_delete_folder)
```

In the above code, we first use the os.listdir() function to get a list of all the files in the download folder. We then use the os.path.join() function to create the full path to each file. We check if the file is a regular file using the os.path.isfile() function, and if it is, we check if the last modification time of the file is older than 30 days. If it is, we move the file to the to\_delete folder using the shutil.move() function.

**Step 4: Put It All Together**

Putting all the code together, we get:

```python
import os
import shutil
import time

download_folder = "[path to download folder]"
to_delete_folder = "[path to to_delete folder]"

for file in os.listdir(download_folder):
    path = os.path.join(download_folder, file)
    if os.path.isfile(path):
        if os.path.getmtime(path) < (time.time() - 30 * 86400):
            shutil.move(path, to_delete_folder)
```

Save the above code as, for example, “clean-downloads.py” and run it to scan your download folder and move files older than 30 days to the to\_delete folder.

**Conclusion**

I wrote this script using WSL on Windows, and used the Linux path format; e.g. “/home/username/Downloads”. You could use the same method or, if you prefer, use the Windows format. You would only need to ensure that you use double back slashes “\\\\”; e.g “C:\\\\Users\\\\SamSpade\\\\Downloads\\\\”.

I hope you find this useful. #HappyCoding
