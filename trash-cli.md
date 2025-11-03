Title: I Deleted the Wrong Files (Again) — So I Built a Safer Trash CLI for Linux
Date: 2025-10-31 18:50
Tags: Linux, DevOps, English
Category: 系统运维
Slug: trash-cli
Author: muxueqz
--------------------------------------------------------------------------

If you’ve ever worked on Linux long enough, you know this story:  
you run a quick `rm -rf something/`, and only afterward realize… it was the *wrong* directory.

That happened to me (again). I tried to recover the data using `xfs_undelete`, but as often with XFS, the recovery failed — the deleted files were gone forever.

So instead of relying on recovery tools, I decided to **make deletion itself safer**.  
That’s how my small script, [`trash.sh`](https://github.com/muxueqz/trash.sh), was born.

---

### 🗑️ What `trash.sh` Does

`trash.sh` is a lightweight Bash script that replaces risky `rm` operations with something safer.  
Instead of permanently deleting files, it moves them to a **`.trash` directory on the same partition**, appending a timestamp to avoid name conflicts.

Example:

```
./trash.sh /data/myfile.txt

```

➡ Moves it to  
`/data/.trash/myfile.txt-2025-10-31_12-00-00`

This way, every filesystem manages its own trash folder — simple, fast, and cross-partition safe.

---

### 🧭 Why a Per-Partition Trash Folder?

Desktop environments like GNOME or KDE use a central `~/.local/share/Trash` folder.  
That works fine for GUI apps, but if you’re moving files across partitions in the terminal, it can fail with *“Invalid cross-device link”* errors.

`trash.sh` solves that by always keeping trash **on the same mount point** as the original file.  
That means:

* The `mv` operation never crosses devices.
* No root or permission errors.
* Each disk or mount can be cleaned up independently.

---

### ⚙️ Usage Overview

#### 1. Move Files or Folders to Trash

```
./trash.sh <file_or_directory>

```

Moves the target(s) into a `.trash` folder located on the same partition, tagging each with a timestamp.

---

#### 2. Clear All Trash Folders

```
./trash.sh -c

```

Searches for `.trash` directories on all mounted partitions, displays what it finds, and asks for confirmation before deleting their contents.

---

#### 3. Clear Trash on a Specific Partition

```
./trash.sh -c /data

```

Clears only `/data/.trash`, showing:

* Number of files
* Number of directories
* Total size

Then prompts for confirmation before cleanup.

---

### 🧩 Features at a Glance

✅ Safer alternative to `rm`  
✅ Works per partition — no cross-device issues  
✅ Preserves ownership and permissions  
✅ Interactive confirmations before clearing  
✅ Minimal and dependency-free

---

### 🚀 Installation

1. Clone it from GitHub:

   ```
   git clone https://github.com/muxueqz/trash.sh
   cd trash.sh
   chmod +x trash.sh

   ```
2. (Optional) Add it to your `$PATH` or alias it:

   ```
   alias rm='trash.sh'

   ```

Now every time you “delete” something, it’s moved to a safe `.trash` directory instead of being permanently erased.

---

### 💡 Why I Made This

After losing important files more than once, I realized two things:

1. Backups are great — but not always up to date.
2. Recovery tools like `xfs_undelete` can’t save you every time.

So I built `trash.sh` to add a **simple safety layer** between me and permanent deletion.  
It’s not complex, it doesn’t depend on desktop systems, and it works anywhere Bash does.

Sometimes, the best tools are the small ones that prevent big mistakes.
