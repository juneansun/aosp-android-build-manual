# aosp-android-build-manual

1. 소스코드 다운로드

The Android source tree is located in a Git repository hosted by Google. The Git repository includes metadata for the Android source, including changes to the source and when the changes were made. This page describes how to download the source tree for a specific Android code line.

To start with a factory image for a specific device instead of downloading the source, see [Selecting a device build](https://source.android.com/docs/setup/build/running#selecting-device-build).

## Initializing a Repo client

After [installing the Repo Launcher](https://source.android.com/docs/setup/develop#installing-repo), set up your client to access the Android source repository:

1.  
    
    Create an empty directory to hold your working files. Give it any name you like:
    
    ```
    mkdir WORKING_DIRECTORYcd WORKING_DIRECTORY
    ```
    
2.  
    
    Configure Git with your real name and email address. To use the Gerrit code-review tool, you need an email address that's connected with a [registered Google account](https://www.google.com/accounts). Ensure that this is a live address where you can receive messages. The name that you provide here shows up in attributions for your code submissions.
    
    ```
    git config --global user.name Your Namegit config --global user.email you@example.com
    ```
    
3.              
    
    Run `repo init` to get the latest version of Repo with its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source are placed within your working directory.
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest
    
    ```
    
    To check out the main branch:
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest -b main
    
    ```
    
    To check out a branch *other than main*, specify it with `-b`. For a list of branches, see  [Source code tags and builds](https://source.android.com/docs/setup/about/build-numbers#source-code-tags-and-builds).
    
    ***For Python 2***
    
    **Warning:** Python 2 support was sunset on Jan 1, 2020 as detailed in [Sunsetting Python 2](https://www.python.org/doc/sunset-python-2/). All major Linux distributions are deprecating support for the Python 2 package. Google strongly recommends that you migrate all your scripts over to Python 3.
    
    **Note:** AOSP ships with its own copies of the Python 2 and Python 3 packages, and you can use the version (such as [SEPolicy](https://android.googlesource.com/platform/system/sepolicy/+/6e60287a1f73c4f792350f7c86dc7bb3e1d6d623/build/Android.bp)) that's included in the source tree. Google is migrating all scripts in the Android source tree to Python 3, and the embedded copy of Python 2 might be deprecated. 
    For additional details, see [Porting Python 2 Code to Python 3](https://docs.python.org/3/howto/pyporting.html), and [advice on sunsetting your Python 2 code.](https://python3statement.org/practicalities/)
    
    ***For Python 3***
    
    If you get a "`/usr/bin/env 'python' no such file or directory`" error message, use one of the following solutions:
    
    If your Ubuntu 20.04.2 LTS is a newly installed (vs. upgraded) Linux version:
    
    ```
    sudo ln -s /usr/bin/python3 /usr/bin/python
    ```
    
    If using Git version 2.19 or greater, you can specify `--partial-clone` when performing `repo init`. This makes use of Git's [partial clone](https://crbug.com/git/2) capability to only download Git objects when needed, instead of downloading everything. Because using partial clones means that many operations must communicate with the server, use the following if you're a developer and you're using a network with low latency:
    
    ```
    repo init -u https://android.googlesource.com/platform/manifest -b main --partial-clone --clone-filter=blob:limit=10M
    
    ```
    
    For Windows OS only: if you get an error message stating that symbolic links couldn't be created, causing `repo init`to fail, reference the GitHub [Symbolic Links documentation](https://github.com/git-for-windows/git/wiki/Symbolic-Links) to create these, or to enable their support. For non-administrators, see the [Allowing non-administrators to create symbolic links](https://github.com/git-for-windows/git/wiki/Symbolic-Links#creating-symbolic-links) section.
    

A successful initialization ends with a message stating that Repo is initialized in your working directory. Your client directory now contains a `.repo` directory where files such as the manifest are kept.

## Downloading the Android source tree

To download the Android source tree to your working directory from the repositories as specified in the default manifest, run:

```
repo sync
```

To speed syncs, pass the `-c` (current branch) and `-jthreadcount` flags:

```
repo sync -c -j8
```

The Android source files are downloaded in your working directory under their project names.

To suppress output, pass the `-q` (quiet) flag. See the [Repo Command Reference](https://source.android.com/docs/setup/create/repo) for all options.
