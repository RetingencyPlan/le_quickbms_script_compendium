this is a list of stuff i have yet to release as scripts due to said scripts needing something else to be barely passable

# THE BIG NEWS

- tri-ace - PS1-PS2 era
  - this is the era in which all tri-ace games had an huge, monstruous archive file to take care of everything within the disc. how exactly did that archive work varied between these two platforms.
    - (PS1) SO2.BIN and VALKYRIE.BIN employ a weird header that takes advantage of its CD-ROM format to locate its own files through CD-ROM sector addresses, further compounded by an weird encryption algorithm that goes like this
      - the encryption/decryption process of these archives involve a for-next looping cycle that stops with a certain number[1] and within that cycle you need to have at least two variables so you can get some bytes at different positions[2](32-bit variable in one position and 8-bit variable in another). this is outlined as two steps:
        - (1/2) for the first 32-bit variable you need to have a **seed-numbering variable** *left-shift* some **crucial number**[3] by 1 then *xor* that 32-byte variable(using itself as a source) with that exact crucial number, then *xor* said crucial number(using itself as a source) with that seed-numbering variable at which point the process will then move on to that 8-bit variable that gets a byte from a certain position. 
        - (2/2) from there on, that seed-numbering variable *complements*(NOR in mathematical terms, kinda like "*xor* 0xffffffff") the crucial number, the crucial number is *xor*-ed with that 8-bit variable that (like the 32-bit variable "explained" above) just gets the data within a certain bit. from there on another variable just takes that crucial number and focuses only the last byte in said number for future use, and finally the crucial number uses a seed-numbering variable alongside its **clone number**[4] and *xors* them together.
        - is that tl;dr not enough for you!? then witness this monster in action!
          ```
          for i = 0 < index_count
            goto pos_01
            get byte_01 long
            math pos_01 + 4
            xmath seed_number "crucial_number << 1"
            math byte_01 ^ crucial_number
            math crucial_number ^ seed_number
            goto pos_02
            get byte_02 byte
            math pos_02 + 1
            math seed_number ~ crucial_number
            math crucial_number ^ byte_02
            xmath crucial_number "seed_number ^ clone_number"
          next i
          ```
      - the actual header(post-decryption process) goes like this:
        - (1) the first 4 bytes of the archive is a literal **signed archive number**[5], surely not something you want to locate the CD-ROM sector in which the file is located and calculate the total number of sectors that consist of an entire size for that file with so this number is skipped entirely.
        - (2) from there on the header consists of two parts :
          - the first part is a listing of all the file offsets and it's located right after that signed archive number.
            - the format in which a file offset is stored upon is a based on a CD-ROM sector address. that means you get at least four bytes that consists of hour, minute, second and... the last byte of a total number of sectors for a single file. in that order.
            - if, for some reason, said variable's value is absolutely zero this value can just be ignored.
          - the second part is at listing of all the first bytes of a total number of sectors for every single file.
            - as the second part is also tied to the first part, this can also mean that this 8-bit variable will contain a zero value for consistency sake.
          - for consistency sake, the way these two parts are organized remain the same. what this means is if a file offset value is simply zero then the second part follows suit.
        - (3) the way a file offset is located is pretty simple, although in my humble opinion it's not quite sane - look for a CD-ROM sector address that has the exact hour, minute and second in which the file is actually located - and the file size is calculated as something along the lines of "pick the first byte of a total number of sectors for a single file, left-shift said byte by 8, then pick the last byte of a total number of sectors for a single file and combine them together through *or*", from there on the file is being passed through with said number of sectors in which encompasses the entire size of a single file.
    - (PS2) sotet.bin, radiata.bin and vp2.bin is basically a successor to the above archive format that was actually used in tri-ace's PS1 games, only this time it takes advantage of the fact that the archive is stored on a DVD disc to locate a bunch of files through DVD sector addresses, not to mention the above encryption algorithm has been fleshed out to hell and back... with an vengeance!
      - the encryption/decryption process of these archives is actually *quite* sophisticated here. outlining all of this as steps here is not that simple, and since this was adapted from one of those no-named functions out of the main exe of Star Ocean Till the End of Time that meant a lot of redundant stuff had to be simplified into what you're seeing/reading right now.
        - (1/2) the decryption process starts with a for-next looping cycle that ends with a certain number[6], and within that cycle is *another* for-next looping cycle that ends with 4... now *that is* ***where the fun really begins***
        - (2/2) within that cycle you need to get three parts of data through three 32-bit variables stored at different positions[7], all while trying to keep up the rhythm of the encryption/decryption action that keeps happening *one part at a time*. mere words will never be enough to describe these fast-paced action scenes, but i'll do my best.
          - (1/4) first part - have the **first seed-numbering variable** grab a **crucial number**[8] and *left-shift* said number by 1, *xor* the first 32-bit variable that grabs the header data from a certaion position(using itself as a source) with that same crucial number, then *xor* said crucial number(using itself as a source) with that seed-numbering variable.
          - (2/4) second part - *complement*(see the explanation above that goes after complements) that seed-numbering variable with a crucial number then have the **second seed-numbering variable** pick up the first one with a clone number[4] then have the **first seed-numbering variable** *left-shift* the **second seed-numbering variable** by 2, *xor* the **second seed-numbering variable**(using itself as a source) with said clone number, then *xor* the second 32-bit variable that grabs the header data from a certaion position(using itself as a source) with that same crucial number.
          - (3/4) third part - *xor* the third 32-bit variable that grabs the header data from a certaion position(using itself as a source) with that same crucial number, and and just *xor* the **second seed-numbering variable**(using itself as a source) with the **first seed-numbering variable**
          - (4/4) now have all three parts be done ***twice*** and exchange these same vaiables between them and this, ladies and gentlemen, is how you decrypt the header!
          - is all that tl;dr *still* not enough for you!? well then, see this monster in action!
            ```
            xmath full_count "index_count >> 3"
            for i = 0 < full_count
              for j = 0 < 4
                goto pos_01
                get byte_01 long
                math pos_01 + 4
                xmath first_seed_number "crucial_number << 1"
                math byte_01 ^ crucial_number
                math crucial_number ^ first_seed_number
                goto pos_02
                get byte_02 long
                math pos_02 + 4
                math first_seed_number ~ crucial_number
                xmath second_seed_number "first_seed_number ^ clone_number"
                xmath first_seed_number "second_seed_number << 2"
                math first_seed_number ^ clone_number
                math byte_02 ^ crucial_number
                goto pos_03
                get byte_03 long
                math pos_03 + 4
                math byte_03 ^ second_seed_number
                math second_seed_number ^ first_seed_number
                goto pos_01
                get byte_04 long
                math pos_01 + 4
                xmath first_seed_number "second_seed_number << 1"
                math byte_04 ^ second_seed_number
                math second_seed_number ^ first_seed_number
                goto pos_02
                get byte_05 long
                math pos_02 + 4
                math first_seed_number ~ second_seed_number
                xmath crucial_number "first_seed_number ^ clone_number"
                xmath first_seed_number "crucial_number << 2"
                math first_seed_number ^ clone_number
                math byte_05 ^ second_seed_number
                goto pos_03
                get byte_06 long
                math pos_03 + 4
                math byte_06 ^ crucial_number
                math crucial_number ^ first_seed_number
              next j
            next i
            ```
      - the actual header(post-decryption process) goes like this:
        - (1) the first 4 bytes of the header consist of a **signed archive number**[9] that varies from archive to archive, same as the PS1 games outlined above. again, this part will be ignored as the first "file" is pretty much the (encrypted) header itself. yes, even the archive header *itself* is referenced in two of all three parts that's been outlined below.
        - (2) from there on the header consists of three parts:
          - the first part is a listing of all located file offsets.
            - the format in which a file offset value is stored on is based on a DVD sector address.
          - the second part is a listing of all file sizes.
            - the format in which a file size value is stored on is the same as the first part.
          - the third part is basically the same as the first one except it's now possessing an "anything that's engine-relevant" mentality, meaning it acts as if only these kinds of files were *the ones* that were being stored in the archive rather than all kinds of files.
            - it should be no surprise as to the kind of format this part stores.
        - (3) thanks to the simplified format of all three parts, you just have to left-shift all these values by 11 for a quick extraction process.
  - [1] 0x1400 for SO2.BIN, 0x1200 for VALKYRIE.BIN
  - [2] for a 32-bit byte-getting variable this position goes right when the archive actually starts, although for a 8-bit byte getting variable it's 0x4800 for SO2.BIN, 0x5000 for VALKYRIE.BIN
  - [3] crucial number is 0x13578642 for SO2.BIN, 0x64283921 for VALKYRIE.BIN
  - [4] clone number is exactly like the crucial number
  - [5] signed archive number is 0x34829314 for SO2.BIN, 0x19563824 for VALKYRIE.BIN
  - [6] 0x1400 for sotet.bin, 0x1200 for radiata.bin, 0xc00 for vp2.bin - all of them that shall be *right-shifted* by 3 instead of having its current looping number be *added* by 8 as the main exe for either of these games said archives were stored on would much rather say
  - [7] as we're talking about three 32-bit variables here, the position for the first one is always 0, the position for the second one is 0x5000 for sotet.bin, 0x4800 for radiata.bin, 0x3000 for vp2.bin, and the position for the third one is 0xa000 for sotet.bin, 0x9000 for radiata.bin, 0x6000 for vp2.bin
  - [8] crucial number is 0x13578642 for *both* sotet.bin and radiata.bin(same number as for SO2.BIN, see [3] above), 0x49287491 for vp2.bin
  - [9] signed archive number is 0x00000400 for sotet.bin, 0x34829314 for radiata.bin(same number as for SO2.BIN, see [5] above), 0x18471208 for vp2.bin
  
- PlayStation (1994) - MDEC
  - yes.

Others to follow...
