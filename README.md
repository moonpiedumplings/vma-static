Proxmox uses a custom formats for their backups/exported VM's called "vma". It's usually compressed via zst, in a zstd file. 

Their tool is custom, and not packaged anywhere else outside of Promox, so this is how I extract it to get it on other Linux distros. 

These steps might have to be adapted if you are on Arm, but the overall process should work the same. Also, this probably only works on Linux, as I don't think Proxmox has any builds or tooling for Mac.

First, use a Proxmox docker image to get the tool:

`podman run -it --rm --entrypoint /bin/bash docker.io/dockurr/proxmox`

I use podman here but docker should work well. 

Next up, use wget to get this tool, [sharun](https://github.com/VHSgunzo/sharun) into the Proxmox docker container. Sharun is a tool that can convert a dynamically linked library into a static binary.

Inside the proxmox docker container:

```
# You should already be in /usr/local/bin, but you might have to move the binary there or to another spot into the path if that's not the case

wget 'https://github.com/VHSgunzo/sharun/releases/download/v0.8.1/sharun-x86_64-aio' -O sharun


chmod +x sharun

# This extracts vma to a static binary in your current directory

sharun lib4bin --with-wrappe --dst-dir . /usr/bin/vma 
```

Outside of the Proxmox docker container, you can use `docker/podman cp` to copy the binary out. 

```
[moonpie@nefertem vma-static]$ podman cp busy_mclean:/usr/local/bin/vma .
[moonpie@nefertem vma-static]$ ls -lh vma 
-rwxr-xr-x 1 moonpie moonpie 24M Oct 12 14:02 vma
[moonpie@nefertem vma-static]$ ldd vma 
        statically linked
[moonpie@nefertem vma-static]$ ./vma --help
usage: vma command [command options]

vma list <filename>
vma config <filename> [-c <config>]
vma create <filename> [-c <config>] [-d format=<format>:<device name>=<path> [-d ...]]
vma extract <filename> [-d <drive-list>] [-r <fifo>] <targetdir>
vma verify <filename> [-v]
```

Tada! Then you can just drop this in your path somewhere in order for it to be usable to extract VM's. 

Now, I'm going to put this x86-64 binary in the github releases so I can reuse it easily in the future. I put the steps here due to supply chain issues 

This repo also doesn't have a license, so I would attribute all code to the license of the various projects of the original owner.
