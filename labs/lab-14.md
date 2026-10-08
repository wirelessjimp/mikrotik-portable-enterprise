# Lab 14 — File Transfer

*Prerequisites: Lab 0 (the portal), Lab 4 (your USB drive is formatted and shows up as `usb1`)*

**Why:** Network gear often needs a TFTP server, for firmware images and configuration files. MikroTik has one built in, and it can serve files from your USB drive without a laptop attached. In this lab you serve one test file from your L009 over TFTP and fetch it back. Then you turn on FTP and move files to and from your USB drive, including a large one.

### 14.1 Get the test file

1. On the portal, in the **Downloads** card, click **tftp-test.txt**. Look near the address bar for **Insecure download blocked**, and click **Keep**.
2. Open your **Downloads** folder. The file `tftp-test.txt` is 128,159 bytes (Finder shows about 128 KB).
3. Note its fingerprint, so you can check later that the copy you fetch is identical. In a terminal on your laptop, run the command for your system:

   **macOS:**

```
shasum -a 256 tftp-test.txt
```

   **Windows:**

```
certutil -hashfile tftp-test.txt SHA256
```

   The fingerprint ends in `da74d78c`. The full value is:

```
28f66f7377aea106ef265e37fb19e321349fc612811a117efd5578d9da74d78c
```

### 14.2 Put it on the USB drive

4. **L009 window:** click **New Terminal** and run `/disk/print`. `usb1` is in the list, mounted.
5. Click **Files**, then use the upload button and pick `tftp-test.txt`. It lands in the root of **Files**.

   > **Note:** Dragging a file from your computer into WinBox doesn't work. Use the upload button.

6. In the **Files** list, drag `tftp-test.txt` onto the `usb1` folder.
7. In the Terminal, run:

```
/file/print where name~"tftp"
```

The list shows `usb1/tftp-test.txt`, `.txt file`, about `125.2KiB`.

### 14.3 Turn on the TFTP server

8. Click **IP**, then **TFTP**, then **New**. Set:
   - **Req. Filename:** `tftp-test.txt` (the name a client asks for)
   - **Real Filename:** `usb1/tftp-test.txt` (where the file really is, with no leading slash)
   - **Allow:** checked
   - **Read Only:** checked
   - **IP Addresses:** leave blank, so any client can read it

   Click **Apply**, then **OK**.

9. In the Terminal, run:

```
/ip/tftp/print
```

The entry shows `tftp-test.txt` mapped to `usb1/tftp-test.txt`, **Allow** `yes`, **Read Only** `yes`, and **Hits** `0`.

> ### ⚠️ STOP AND READ
> TFTP has no login. Anyone who can reach your L009 can read every file you map. Keep **Read Only** checked, and map only files you mean to share.

### 14.4 Fetch the file from your laptop

10. **macOS:** open a terminal on your laptop. Use a fresh folder, so the download doesn't mix with the file in **Downloads**:

```
mkdir -p ~/tftp-got && cd ~/tftp-got
tftp 192.168.88.1
```

11. At the `tftp>` prompt, type `get tftp-test.txt`. It prints `Received 128159 bytes during 0.1 seconds in 251 blocks`. Type `quit`.

    > **Why:** TFTP sends a file in 512-byte blocks, and 128,159 bytes comes to 251 of them.

12. Check the copy:

```
ls -l tftp-test.txt
shasum -a 256 tftp-test.txt
```

The size is `128159` and the fingerprint matches the one in step 3, all 64 characters. A match means the file arrived intact.

13. **L009 Terminal:** run `/ip/tftp/print`. **Hits** now reads `1`. The server counts each request it answers.

> **Note:** `192.168.88.1` is your L009's backdoor address. Use the address of whichever network your laptop is on.

### 14.5 Turn on the FTP service

FTP moves files in both directions and handles large files that WinBox's upload button struggles with. Your L009's setup turns the FTP service off, so you turn it on.

14. **L009 window:** click **IP**, then **Services**. Double-click **ftp**. It shows as disabled.
15. Check **Enabled**, leave **Port** at `21`, and set **Available From** to `192.168.88.0/24`. Click **OK**.
16. In the Terminal, run:

```
/ip/service/print where name=ftp
```

The line shows `21`, `tcp`, and `192.168.88.0/24`, and has no **X** in front of it.

> **Note:** FTP sends passwords in plain text. **Available From** limits who can reach it, so keep it to your own network.

### 14.6 Create two FTP users

You make one user who can only read files and one who can also write. Writing is how large files get onto your USB drive.

17. Click **System**, then **Users**, then the **Groups** tab, then **New**. Set **Comment** to `FTP Write`. Set **Name** to `ftp-write`. Under **Policies**, check `ftp`, `read`, and `write`, and leave every other policy unchecked. Click **Apply**, then **OK**.
18. Click **New** again. Set **Comment** to `FTP Read`. Set **Name** to `ftp-read`. Check `ftp` and `read` only. Click **Apply**, then **OK**.
19. Click the **Users** tab, then **New**. Set **Name** to `ftpwrite`, **Group** to `ftp-write`, and **Allowed Address** to `192.168.88.0/24`. Enter a throwaway password in **Password** and **Confirm Password**. Click **Apply**, then **OK**.
20. Click **New** again. Set **Name** to `ftpread`, **Group** to `ftp-read`, and **Allowed Address** to `192.168.88.0/24`. Enter a different throwaway password. Click **Apply**, then **OK**.
21. Record both passwords in **Lab Notes**.
22. In the Terminal, run:

```
/user/group/print where name~"ftp"
/user/print where name~"ftp"
```

Both groups show their policies, with every unchecked policy marked `!`, such as `!winbox`, `!ssh`, and `!policy`. `ftp-read` also shows `!write`. Both users are listed with their groups and `192.168.88.0/24`.

### 14.7 Test the read-only user

Run the next commands in a terminal on your laptop, in the folder where you saved `tftp-test.txt` in 14.4 (`tftp-got`). Each command asks for the password.

> **Note:** **Windows:** type `curl.exe`, not `curl`. In PowerShell, `curl` is a different command.

23. List your USB drive:

```
curl --user ftpread ftp://192.168.88.1/usb1/
```

The listing shows `tftp-test.txt`, and the container folders from Lab 4 (`nginx`, `speedtest`, `iperf3`, and others), owned by `root`.

24. Download the test file, and check it:

```
curl --user ftpread -o ftp-got.txt ftp://192.168.88.1/usb1/tftp-test.txt
shasum -a 256 ftp-got.txt
```

The fingerprint matches the one in step 3.

25. Try an upload. Make a small file first:

```
printf 'ftp upload test\n' > ftp-up.txt
curl --user ftpread -T ftp-up.txt ftp://192.168.88.1/usb1/
```

It fails with `curl: (25) Failed FTP upload: 550`. The read-only user can't write.

### 14.8 Test the write user

26. Upload the same file with the write user:

```
curl --user ftpwrite -T ftp-up.txt ftp://192.168.88.1/usb1/
```

It shows a progress line and no error.

27. **L009 Terminal:** run:

```
/file/print where name~"ftp-up"
```

The list shows `usb1/ftp-up.txt`, 16 bytes.

### 14.9 Try a large file (macOS and Linux)

28. Make a 10 MB file of random data. Type the file name without `~/`:

```
dd if=/dev/urandom of=ftp-test-10mb.bin bs=1048576 count=10
```

It ends with `10485760 bytes transferred`.

   > **Note:** The shell doesn't expand `~` after `of=`. A name like `~/tftp-got/file` fails with `No such file or directory`.

29. Upload it with the write user:

```
curl --user ftpwrite -T ftp-test-10mb.bin ftp://192.168.88.1/usb1/
```

The progress line shows a `10.0M` total and about 20 MB per second on the instructor's router.

30. **L009 Terminal:** run `/file/print where name~"usb1/ftp-test"`. The list shows `usb1/ftp-test-10mb.bin` at `10.0MiB`.

    > **Note:** Include `usb1/` in the pattern. Without it, `ftp-test` also matches `tftp-test.txt`.
31. Pull it back with the read-only user and compare:

```
shasum -a 256 ftp-test-10mb.bin
curl --user ftpread -o ftp-back.bin ftp://192.168.88.1/usb1/ftp-test-10mb.bin
shasum -a 256 ftp-back.bin
```

The two fingerprints are identical.

32. **L009 Terminal:** remove the test files from the drive:

```
/file/remove [find where name~"usb1/ftp-"]
```

> ### ⚠️ STOP AND READ
> The read-only user can still read everything on the drive, and probably more of the router's files. I only listed `usb1/`. Don't give out `ftpread` outside the lab, and don't leave exported certificates or keys in **Files**.
