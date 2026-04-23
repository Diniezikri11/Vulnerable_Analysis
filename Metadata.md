# Week 4: METADATA

**Tools learned:**
1. `file`
2. `strings`
3. `exiftool`
4. `hexeditor`
5. `binwalk`

## Task

| Picture        | Tools             | Link / Command       | POC  | Analysis |
|----------------|-------------------|----------------------|------|----------|
| Ocean.jpg      | exiftool          | https://exif.tools/  | <img width="2439" height="1503" alt="image" src="https://github.com/user-attachments/assets/8fa9fdf8-1fd7-4c0c-b720-068518035a29" />| Online tools can also output same result as command in Kali          |
|                |                   | `exiftool ocean.jpg` | <img width="1384" height="863" alt="image" src="https://github.com/user-attachments/assets/37809627-d718-4639-b377-26f254bdaf8f" />| Can clearly saw the flag in comment section          |
| computer.jpg   | strings           | https://www.dcode.fr/strings-extractor | <img width="1940" height="1384" alt="string web computer jpg" src="https://github.com/user-attachments/assets/b1f67ba1-4290-4350-9776-ea9cf127ae44" />| online tools for command `strings` |
|                |                   | `strings computer.jpg` | <img width="1378" height="612" alt="string computer jpg" src="https://github.com/user-attachments/assets/dfdf2774-695e-4f39-8cf9-a4cd2d7b0aa3" />| used to extract human-readable text from files, especially binary files.|
| dog.jpg        | binwalk           | `binwalk dog.jpg` <br> `binwalk -e dog.jpg` <br> `cd _dog.jpg.extracted/` | <img width="1382" height="1030" alt="dog" src="https://github.com/user-attachments/assets/752bd906-f219-4146-9258-03775f11fcab" />| `binwalk` usually use for finding hidden file in the main file  |
| solitaire.exe  | file | `file solitaire.exe` | <img width="1384" height="112" alt="solitaire exe" src="https://github.com/user-attachments/assets/dbcb7eda-e1af-40ce-b23b-63505d672eb8" />| solitare.exe is actually a PNG file |
| rubiks.jpg     | file | `file rubiks.jpg`    | <img width="1387" height="72" alt="image" src="https://github.com/user-attachments/assets/37a6a057-419d-4190-9fd2-d41b34e62061" />| After checking the jpg file is actually a png |
