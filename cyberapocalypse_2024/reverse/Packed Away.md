By doing strings, we can see this:
```bash
$ strings packed | grep }
Hr3t_0f_th3_p45}
slIT$}
}aw993u
```

The first one looks like a flag part (of the ...)

I also found this string within the binary:
```
This file is packed with the UPX executable packer http://upx.sf.net
```

I'm going to unpack it using UPX:

```bash
./upx -d ~/Documents/ctf/rev_packedaway/packed 
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jan 3rd 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     22867 <-      8848   38.69%   linux/amd64   packed

Unpacked 1 file.
```

Doing another strings on the binary gives us the flag !

```bash
$ strings ./packed | grep HTB{
HTB{unp4ck3dr3t_HH0f_th3_pH0f_th3_pH0f_th3_pH0f_th3_pH
HTB{
HTB{unp4ck3d_th3_s3cr3t_0f_th3_p455w0rd}

```
