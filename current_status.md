this is a list of stuff i have yet to release as scripts due to said scripts needing something else to be barely passable

# THE BIG NEWS

- tri-ace - PS1-PS2 era
  - (PS1) SO2.BIN and VALKYRIE.BIN employ a weird header that takes advantage of its CD-ROM format to locate its own files through CD-ROM sectors, compounded further by an weird encryption algorithm that goes like this
    - the encryption/decryption process of these archives involve a for-next looping cycle that stops with a certain number[1] and within that cycle you need to have at least two variables so you can get some bytes at different positions[2](32-bit variable in one position and 8-bit variable in another). this is outlined as two steps:
      - (1/2) for the first 32-bit variable you need to have *another* **"seed-numbering" variable** *left-shift* some **crucial number**[3] by 1 then *xor* that 32-byte variable by that exact crucial number, then *xor* said crucial number with that "seed-numbering" variable at which point the process will then move on to that 8-bit variable that gets a byte from a certain position. 
      - (2/2) from there on, that "seed-numbering" variable "complements"(NOR in mathematical terms, kinda like "*xor* 0xffffffff") the crucial number, the crucial number is *xor*-ed with that 8-bit variable that (like the 32-bit variable "explained" above) just gets the data within a certain bit. from there on another variable just takes that crucial number and focuses only the last byte in said number for future use, and finally the crucial number uses a "seed-numbering" variable alongside its **clone number**[4] and *xors* them together.
    - from there on the actual header goes like this
      - (1) the first 4 bytes of the archive is a literal **signed archive number**[5], surely not something you want to locate the CD-ROM sector in which the file is located and calculate the total number of sectors that consist of an entire size for that file with so this number is skipped entirely
      - (2) from there on the header goes into two parts 
        - the first part is the CD-ROM sector location of the file and it's located right after that signed archive number. it's a single 32-bit variable that contains at least one byte of info that goes like this
          - hour, minute, second and... the last byte of a total number of sectors for a single file
          - if, for some reason, said variable's value is absolutely zero this value can just be ignored
        - the second part is where the you meet all the first bytes of a total number of sectors for every single file. it's located right after the first part
          - as the second part is also tied to the first part, this can also mean that this 8-bit variable will contain a zero value for consistency sake
      - (3) the way a file offset is located is pretty simple - look for a CD-ROM sector that has the exact hour, minute and second in which the file is actually located - and the file size is calculated as something along the lines of "pick the first byte of a total number of sectors for a single file, left-shift said byte by 8, then pick the last byte of a total number of sectors for a single file and combine them together through *or*", from there on the file is being passed through with said number of sectors in which encompasses the entire size of a single file and this is how a game treats its files
    - [1] for-next looping cycle that ends with number 0x1400 for SO2.BIN and 0x1200 for VALKYRIE.BIN
    - [2] for a 32-bit byte-getting variable this position goes right when the archive actually starts, although for a 8-bit byte getting variable it's either 0x4800 for SO2.BIN or 0x5000 for VALKYRIE.BIN
    - [3] crucial number is 0x13578642 for SO2.BIN and 0x64283921 for VALKYRIE.BIN
    - [4] clone number is exactly like the crucial number
    - [5] signed archive number is 0x34829314 for SO2.BIN and 0x19563824 for VALKYRIE.BIN
  - (PS2) sotet.bin, radiata.bin and (insert Valkyrie Profile 2: Silmeria's .bin filename here) is basically a successor to the above archive format that was actually used in tri-ace's PS1 games, only this time it takes advantage of the fact that the archive is stored on a DVD disc to locate a bunch of files through DVD sectors, not to mention the above encryption algorithm has been fleshed out to hell and back
    - the actual header(post-decryption process) has yet to be figured out - all i know is that said header consists of three parts, the third of which i have yet to find out, as for the rest the first one is a listing of all located DVD sectors of all files(these are basically file offsets acting as if they were on the actual game disc) and the second one is a listing of all encompassed DVD sectors of all files(effectively making them file sizes as a whole)
      - radiata.bin on the other hand uses a signed archive number whose value is the same as SO2.BIN on its (decrypted) header

Others to follow...
