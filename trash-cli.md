Title: I Deleted the Wrong Files (Again) — So I Built a Safer Trash CLI for Linux
Date: 2025-10-31 18:50
Tags: Linux, DevOps, English
Category: 系统运维
Slug: trash-cli
Author: muxueqz
--------------------------------------------------------------------------

If you use Linux long enough, you’ve probably done it too.  
You type a quick `rm -rf something/`, and a few seconds later realize it was the *wrong* folder.

I’ve been there — several times.  
This time, I tried to recover my files using `xfs_undelete`, but as usual with XFS, the deleted data was unrecoverable.  
So I decided to stop trusting recovery tools and instead **make deletion safer** from the start.

That’s how my little script — **`trash.sh`** — was born.

---

### 🗑️ What It Does

Instead of permanently deleting files, `trash.sh` moves them into a **`.trash` directory on the same partition**.  
That means:

* No accidental cross-device moves
* No root permission issues
* Easy cleanup later

Each trashed file is renamed with a timestamp so you can safely trash multiple files with the same name without overwriting.

Example:

```
./trash.sh /data/myfile.txt

```

➡ Moves it to  
`/data/.trash/myfile.txt-2025-10-31_12-00-00`

---

### 🧭 Why a Trash Per Partition?

When I first thought of this, I considered using a global `~/.local/share/Trash` folder (like desktop environments do).  
But that breaks when you delete files on other mounts (like `/mnt/ssd`, `/data`, or `/usbdrive`) — the `mv` command fails across filesystems.

By creating a `.trash` folder **on the same mount point**, `trash.sh` guarantees that:

* The move is instant (same partition).
* You can safely clean up per-disk.
* You avoid cross-device permission headaches.

---

### ⚙️ Basic Usage

#### 1. Move Files to Trash

```
./trash.sh <file_or_directory>

```

Moves the target(s) into a `.trash` directory on the same partition, adding a timestamp to avoid name conflicts.

---

#### 2. Clear All Trash Folders

```
./trash.sh -c

```

Searches for all `.trash` directories across mounted filesystems, shows what it found, and asks for confirmation before clearing everything.

---

#### 3. Clear Trash on a Specific Partition

```
./trash.sh -c /data

```

Clears only the trash folder on the `/data` partition.  
It shows:

* Number of files
* Number of directories
* Total size  
  Then asks for confirmation before deleting.

---

### 🧩 Features at a Glance

✅ Moves files safely instead of deleting them  
✅ Works per partition — no cross-device issues  
✅ Preserves ownership and permissions of `.trash` folders  
✅ Supports clearing all or specific trash locations  
✅ Asks before doing anything destructive

---

### 🔒 Why This Is Better Than Just Using `rm`

`rm` is brutally honest — it deletes things permanently, no second chances.  
`trash.sh` is a simple middle ground: you still work from the terminal, but you get a **layer of safety**.

It’s not bloated, doesn’t need external dependencies, and works even on headless servers where desktop trash systems don’t exist.

---

### 🚀 Installation

1. Save the script as `trash.sh`.
2. Make it executable:

   ```
   chmod +x trash.sh

   ```
3. (Optional) Add it to your `$PATH` or create an alias:

   ```
   alias rm='~/bin/trash.sh'

   ```

Now every time you “delete” something, it just moves to a safe `.trash` folder instead.

---

### 💡 Final Thoughts

After losing files one too many times, I learned two big lessons:

1. Backups are non-negotiable.
2. Prevention is better than recovery.

`trash.sh` isn’t fancy — it’s a few dozen lines of Bash — but it’s saved me countless headaches already.  
Sometimes, the simplest tools are the ones that protect you best.
