# When Hearts Collide

URL: https://tryhackme.com/room/lafb2026e1
Description:
My Dearest Hacker,

Matchmaker is a playful, hash-powered experience that pairs you with your ideal dog by comparing MD5 fingerprints. Upload a photo, let the hash chemistry do its thing, and watch the site reveal whether your vibe already matches one of our curated pups. The algorithm is completely transparent, making every match feel like a wink from fate instead of random swipes.

Come get your dog today!

# Tl, dr:

This room is all about MD5 collision.
-   The server **only uses MD5** to decide if your upload "matches" one of the curated dogs.
-   But it **also prevents trivial replays** by rejecting byte-identical uploads.
-   By uploading a **different file with the same MD5**, you trigger a legitimate match without tripping the deduplication → new random UUID → success redirect → flag revealed.

## Enumeration:
Rustscan:
```
└─$ rustscan -a 10.66.xx.xx -- -A -sV 
Nmap? More like slowmap.🐢

[~] The config file is expected to be at "/home/kali/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.66.xx.xx:22
Open 10.66.xx.xx:80

GoBuster (port 80):
└─$ gobuster dir -w ~/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt -t 100 --timeout 30s -x php,html,htm,js,aspx,txt,bak -r -b 429,404,500,502,503 --url http://10.66.xx.xx 

[+] Url:                     http://10.66.xx.xx
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /home/kali/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt
[+] Negative Status codes:   502,503,429,404,500
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              txt,bak,php,html,htm,js,aspx
[+] Follow Redirect:         true
[+] Timeout:                 30s

Starting gobuster in directory enumeration mode
upload               (Status: 405) [Size: 153]
static               (Status: 403) [Size: 146]
Progress: 498248 / 498248 (100.00%)
Finished
```

## Decision time
Nginx and OpenSSH are both up to date with no current exploitable vulnerabilities. We see an /upload and a /static folder, so let's explore the application.

Hitting both of these with GoBuster showed nothing of value. Time to see what the webpage offers.

## The App
We are presented with a "Doggie dating app" that uses MD5 hashes to find the wet-nosed love of our life. I uploaded a picture of myself:
https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQQo9AfAW59BzmDFRVZw0MWluY53WC_OtTiSA&s

And the upload_success page showed that there were no matches. I noticed that the URL contained what looked like an UUID, not an MD5 hash (Location: /upload_success/dd49de52-03b8-4fc6-xxxx-ddf2fb297292). I took it to CyberChef and it turned out to indeed be a V4 UUID (random.)

I tried an upload again, this time in Burp, to see if I could find an IDOR or the like in the process. Sadly, The app uses a webkit form to upload the picture raw, no processing done that we can see...

Speaking of seeing, I tried to look into the /upload_success/ and /static/ folders. Nginx returned a 403 (kind of expected tbh.)

My first thought at this point was to get an MD5 rainbow table and convert the values to UUID. This is wrongthink - Could you imagine? Even after downloading a several TB file, the overhead on Hydra? My poor laptop! Then I thought about the "featured dog" image on the site.

## Getting closer...
The featured dog image was available at /view/\<UUID>. As a test, I took the UUID I was given for my image and it was available in that directory. So, we have confirmed that there is no pseudo-shenanigans going on here. There really is some kind of UUID matching going on.

This is where I hit a wall. I tried to enumerate the /view/ directory. I tried re-uploading the featured image. I tried a python script that took the UUID of the featured image and get a true MD5 sum from it, then tried matching it to short number strings or common words (flag, password, user, letmein, rockyou...)

Nope, not those.

I took a step back and caffeinated a little. COLLISION of the heart. COLLISION...md5 collisions!
## The solution!
Not going to go super deep into md5 collisions. The short version is md5 is flawed and you can get the same hash value for two different files. Learn to your heart's content here: https://marc-stevens.nl/research/hashclash/

I cloned the Github repo for md5collgen (basically fastcoll, but with a makefile). Running make gave me a binary.

Next, I downloaded the featured dog image (it's available to view, so we are sure to match it...)

Now to make stars collide!

```
└─$ ./md5collgen -p 00795a8b-fb58-47c0-xxxx-af068ddc71b4.jpg -o collision1.bin collision2.bin

Using output filenames: 'collision1.bin' and 'collision2.bin'
Using prefixfile: '00795a8b-fb58-47c0-xxxx-af068ddc71b4.jpg'
Using initial value: d952c4dc451491468eae6178905xxxx

Generating first block: ....
Generating second block: W........................
Running time: 0.894659 s
```

Once this finished, I went back to the upload and presented collision1.bin. It accepted the file, but no match.

Not going to lie, I was a little frustrated. But, I tried collision2.bin and whammo! Flag presented.

This was a classic demonstration of why MD5 is unsuitable for anything security-sensitive involving uniqueness or integrity checks — collisions are practical even against fixed targets when you have a sample of the target file.
