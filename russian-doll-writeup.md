There is a zip-file embedded in the gif which can be extracted with the ```unzip``` command.

The password to each zip is the name of the file, so the first password is "password".

Unzip file with the file name as password iteratively until you reach flag.txt.

Solver script in python:
```
import os
import subprocess


while "flag.txt" not in (curr := os.listdir()):
    for file in curr:
        if file.endswith(".zip"):
            extract_this = file[:-4]
            ending = ".zip"
        elif file.endswith(".gif"):
            extract_this = file[:-4]
            ending = ".gif"
    args = ["unzip", "-q", "-P", extract_this, f"{extract_this}{ending}"]
    subprocess.run(args)
    subprocess.run(["rm", f"{extract_this}{ending}"])
```
