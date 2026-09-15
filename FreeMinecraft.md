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

- `dowloader.sh`

<img width="1845" height="901" alt="image" src="https://github.com/user-attachments/assets/220c1583-a028-474c-8999-f8f456198c68" />

This script set up a de







