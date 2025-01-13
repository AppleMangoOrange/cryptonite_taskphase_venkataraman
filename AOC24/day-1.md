# Day 1

- The website in the challenge is supposedly a YouTube to MP3 converter, but the output is always the same `.zip` file that contains just two files:
  - `song.mp3`
  - `somg.mp3`
- The file `somg.mp3` is not playable, and using `file` reveals that it is a Windows Shortcut file. `exiftool` reveals the command that is run in powershell when the shortcut is clicked, which leads to a ]github script](https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1).
- `Created by the one and only M.M` in the script leads to [this github comment](https://github.com/Bloatware-WarevilleTHM/CryptoWallet-Search/issues/1#issuecomment-2447131903).
