## Challenge Info

> We've received reports that Draeger has stashed a huge arsenal in the pocket dimension Flaggle Alpha. You've managed to smuggle a discarded access terminal to the Widely Inflated Dimension Editor from his headquarters, but the entry for the dimension has been encrypted. Can you make it inside and take control?

## The challenge
My first thought is that we are going to be dealing with some sort of encrypt but lets unzip the file and see what we are dealing with

```bash
% unzip WIDE.zip

Archive:  WIDE.zip
   creating: rev_wide/
[WIDE.zip] rev_wide/wide password: 
  inflating: rev_wide/wide           
  inflating: rev_wide/db.ex          

% cd rev_wide/

% ls

db.ex wide

% 
```

We see that we are dealing with 2 files:
- `db.ex`
- `wide`

Lets see what these files are:

```bash
file db.exe 

db.ex: Matlab v4 mat-file (little endian) , numeric, rows 1835627088, columns 29557

% file wide

wide: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=13869bb7ce2c22f474b95ba21c9d7e9ff74ecc3f, not stripped
```

We see that `db.ex` is some sort of matlab file, more interestingly we see that `wide` is a linux executable. Lets run `strings` on it and see if we can learn anything quickly

```bash
% strings wide  

/lib64/ld-linux-x86-64.so.2
libc.so.6
exit
fopen
ftell
puts
mbstowcs
stdin
printf
strtol
fgets
calloc
fseek
fclose
wcscmp
fread
__cxa_finalize
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
AWAVI
AUATL
[]A\A]A^A_
Which dimension would you like to examine? 
That option was invalid.
[X] That entry is encrypted - please enter your WIDE decryption key: 
[X]                          Key was incorrect                           [X]
Usage: %s db.ex
[*] Welcome user: kr4eq4L2$12xb, to the Widely Inflated Dimension Editor [*]
[*]    Serving your pocket dimension storage needs since 14,012.5 B      [*]
[x] There was a problem accessing your database [x]
[*]                       Displaying Dimensions....                      [*]
[*]       Name       |              Code                |   Encrypted    [*]
[X] %-16s | %-32s | %6s%c%7s [*]
;*3$"
GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7698
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
wide.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
wcscmp@@GLIBC_2.2.5
_ITM_deregisterTMCloneTable
puts@@GLIBC_2.2.5
fread@@GLIBC_2.2.5
stdin@@GLIBC_2.2.5
mbstowcs@@GLIBC_2.2.5
_edata
fclose@@GLIBC_2.2.5
menu
printf@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
fgets@@GLIBC_2.2.5
calloc@@GLIBC_2.2.5
__data_start
ftell@@GLIBC_2.2.5
__gmon_start__
strtol@@GLIBC_2.2.5
__dso_handle
_IO_stdin_used
__libc_csu_init
fseek@@GLIBC_2.2.5
__bss_start
main
fopen@@GLIBC_2.2.5
exit@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```

Looking through the strings we see what looks like a menu and the username of `kr4eq4L2$12xb`. We also see a reference to `db.ex` as it looks to be explaining the usage of the script. Apart from this, nothing else seems to be all that interesting in `wide`. Lets do the same thing with `db.ex`

```bash
% strings db.ex  

Primus
people breathe variety practice
Our home dimension
Cheagaz
scene control river importance
The Ice Dimension
Byenoovia
fighting cast it parallel
The Berserk Dimension
Cloteprea
facing motor unusual heavy
The Hungry Dimension
Maraqa
stomach motion sale valuable
The Water Dimension
Aidor
feathers stream sides gate
The Bone Dimension
Flaggle Alpha
admin secret power hidden
HOt*
0ANe
```

Looks to just be sentences or words. Still don't know too much about these files. Lets try some dynamic analysis and see how this works before jumping into IDA Pro. I have decided to run the file in an ubuntu docker container as its quick and easy to spin up. 

Running `./wide` we get that example usage output that we saw earlier. Lets try running it the way it wants us to.

```bash
% ./wide db.ex

[*] Welcome user: kr4eq4L2$12xb, to the Widely Inflated Dimension Editor [*]
[*]    Serving your pocket dimension storage needs since 14,012.5 B      [*]
[*]                       Displaying Dimensions....                      [*]
[*]       Name       |              Code                |   Encrypted    [*]
[X] Primus           | people breathe variety practice  |                [*]
[X] Cheagaz          | scene control river importance   |                [*]
[X] Byenoovia        | fighting cast it parallel        |                [*]
[X] Cloteprea        | facing motor unusual heavy       |                [*]
[X] Maraqa           | stomach motion sale valuable     |                [*]
[X] Aidor            | feathers stream sides gate       |                [*]
[X] Flaggle Alpha    | admin secret power hidden        |       *        [*]
Which dimension would you like to examine? 1
The Ice Dimension
Which dimension would you like to examine? 2
The Berserk Dimension
Which dimension would you like to examine? 3
The Hungry Dimension
Which dimension would you like to examine? 4
The Water Dimension
Which dimension would you like to examine? 5
The Bone Dimension
Which dimension would you like to examine? 6
[X] That entry is encrypted - please enter your WIDE decryption key: password
[X]                          Key was incorrect                           [X]
```

We now get a prompt to enter a dimension. I iterate through them and the one that peaks my interest is option 6 as its now asking for a decryption key. Maybe we can find something if we decompile the binary. Lets open IDA Pro.

![wide-1.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-1.png)

![wide-2.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-2.png)

![wide-3.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-3.png)

This is clearly where the options are display, we iterate though each option and the counter `var_20` is incremented after every iteration. Once we reach the value in `var_1C`, presumably the number of options available, we jump to another block where a subroutine called `menu` is called.

![wide-4.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-4.png)
Now this is the interesting part of `menu` as we see the same message that we got when we ran the executable `That entry is encrypted - please enter your WIDE decryption key:`. We also see towards the bottom of the block a call to `wcscmp`. I was not familiar with that c call but after a quick google I found it is a function to compare 2 wide strings which are strings with a character size larger than 8 bits. We see that `wcscmp` is comparing the value stored in `s2` to the value stored in the stack variable`pwcs`. Lets see what is stored at the location of `s2`.

![wide-5.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-5.png)
We see the declaration of a wide char with `wchar_t s2` followed by a series of strings. IDA Pro is assuming these are normal size strings but we now know its a wide string. Lets tell IDA its a wide string by clicking on the definition, and click `Option+a`. The following prompt appears.
![wide-6.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-6.png)
If we deselect the UTF-8 we can select several other character encodings. Lets go with the largest `UTF-32LE`. Now this looks really different.

![wide-7.png](https://github.com/kebab01/HTB-Write-Ups/blob/main/images/wide-7.png)
Lets paste into the prompt for the encryption key
```bash
Which dimension would you like to examine? 6
[X] That entry is encrypted - please enter your WIDE decryption key: sup3rs3cr3tw1d3
HTB{som3_str1ng5_4r3_w1d3}
Which dimension would you like to examine? Our home dimension
Which dimension would you like to examine? 
```

YAY!!! it works and reveals the flag.
