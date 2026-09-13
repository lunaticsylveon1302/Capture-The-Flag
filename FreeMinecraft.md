# Free Minecraft

<details>
  
  <summary> <b> Description </b> </summary>

- **Author**: `Devobass`
  
- **Category**: `Forensics - Disk Imaging`
    
- **Tools**: `FTK Imager`
    
- **Files**: `disk.img.xz`,`hash.txt`

</details>

*File > Add Evidence Item > Image File > Next > Browse the source path > Finish*

Upon wandering around in the `home` folder, I found some sussy stuff:

<img width="959" height="562" alt="image" src="https://github.com/user-attachments/assets/293ef349-163e-4bfc-adae-1a163f7b7285" />

Right-click the text and modify the encoding to UTF-8: 

`Ваши файлы зашифрованы, для расшифровки отправьте $36000 на 12345678910 TCB`

Which is translated into:

`Your files are encrypted; to decrypt them, send $36,000 to 12345678910 TCB`. 

Wandering around for more, there is a symlink called `minecrap` containing `/tmp/downloader.sh`

<img width="959" height="562" alt="image" src="https://github.com/user-attachments/assets/34272f18-ade6-4c8f-8487-abde40440f59" />

From these two file, we can infer that our `cool_user` is infected with ransomware while attempting to crack the game Minecraft and downloading from untrusted source from the Internet, I suppose.

I suppose we would like to trace back the downloader. Unfortunately, `/tmp` is empty, since it is self-destructive. But we can look up the user's browser history to retrieve what was lost, I suppose. After doing some research on the whereabouts of browser history on Linux, I found:

<img width="959" height="560" alt="image" src="https://github.com/user-attachments/assets/32fe6bf7-7524-4b7b-96da-8bcb55af3b6e" />

I believe there should be some valuable information around this place:

<img width="959" height="564" alt="image" src="https://github.com/user-attachments/assets/4c941c48-eca5-49db-835a-499766f038f2" />










