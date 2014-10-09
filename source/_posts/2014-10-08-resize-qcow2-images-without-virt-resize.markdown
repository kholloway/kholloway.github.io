---
layout: post
title: "Resize Qcow2 images without virt-resize"
date: 2014-10-08 20:52:54 -0500
comments: true
categories: [Rhel, Qemu, Virt]
---
I'm in the process of switching from kickstart builds for new VM's to image based clones.
I needed to find a simple way to add space to a qcow2 image file along with one of the partitions in it and the volumes it contained.
Most of the guides I found used *virt-resize* to resize the image.
While *virt-resize* makes resizing qcow2 images really easy it has the side affect of making sparse images into non-spare images.
This can end up using a considerable amount of space depending on how much additional space you are adding.
The rest of the options I found have you converting qcow2 to raw and then back again which is slow and also can consume large amounts of space.

[Libguestfs](http://libguestfs.org/guestfs.3.html) to the rescue!

Now there are a few requirements, namely this only works if the partition you want to modify is either the last partition or the only partition and you need to do this all while the VM is shutdown.
In my case it's the last partition I need to expand, the disk layout has boot at partition 1 and partition 2 contains two LVM volumes.
It's both of these volumes that I want to expand via libguestfs.

[full gist available here](https://gist.github.com/kholloway/ded725ea321ce8fe79c7) with some error checking and a few more options.

#### Lets go!

First lets increase the qcow2 image size using the standard *qemu-img* tool, lets add 40GB.

```
qemu-img vmimage.img +40G
```

Now using whatever language you like (libguestfs has lots of bindings for various languages) fire up the guestfs back end.
Note that you can do this in the shell/Bash if you like but calculating the proper partition sector start location is a little harder (not impossible just a bit more work).
Everything below is in Python, copy it to a script or just type along in the Python command line.

```
import guestfs

# For return values we want a dict object
g = guestfs.GuestFS(python_return_dict=True)
g.add_drive_opts('vmimage.img')
g.launch()
```

Lets assume your dev name is **/dev/sda** but you can use the *list_devices* function to find it if you are not sure.
Now lets get the partition list:

```
partitions = g.part_list('/dev/sda')
```

We need the start location of partition 2:

```
part_start = partitions[1]['part_start']
```

Grab the block size of the image:

```
blk_size = g.blockdev_getss('/dev/sda')
```

Calculate the starting sector by dividing our partition start location by the block size:

```
start_sector = part_start / blk_size
```

Now the scary part, we need to delete partition 2:

```
g.part_del('/dev/sda', 2)
```

Ok great the partition is gone, lets create a new one now with the info we gathered above.
The '-1' bit below tells libguestfs that you want the last sector as your end sector.
This is a nice shortcut so you don't need to calculate it yourself:

```
try:
  g.part_add('/dev/sda', 'p', start_sector, -1)
except RuntimeError as msg:
  print "Part add failed due to re-read of partition table, this is normal and expected.. Ignored.."
```

As noted above because the partition table can't be re-read by the kernel you will get an error message.
This is ok, to resolve this lets just stop and restart the libguestfs system to force it to reload the partition table:

```
g.shutdown()
g.add_drive_opts('vmimage.img')
g.launch()
```

Ok now we can resize the PV inside the partition we just expanded:

```
g.pvresize('/dev/sda2')
```

Now in this case I know the exact volume names but if you didn't know it you can use the *lvs* function in libguestfs to list them.
Since I know them already I'm just going to include it in the command below which takes my swap volume from whatever it was (2GB previously) to 6GB now.

```
g.lvresize('/dev/vgroot/swap.vol', 6144)
```

Ok great, swap vol is now 6GB!
Lets expand the root volume with any remaining space in the PV.
This uses a slightly different guestfs command that fills free space with a percentage number.
For my particular use case I just want to fill all remaining space or 100% remaining space as shown below:

```
g.lvresize_free('/dev/vgroot/root.fs', 100)
```

Last steps, lets resize the actual filesystem on the root volume and then check it to make sure everything is OK:

```
g.resize2fs('/dev/vgroot/root.fs')
g.e2fsck('/dev/vgroot/root.fs', correct=True)
```

Now shutdown the guestfs process since we are all done:

```
g.shutdown()
g.close()
```

That's it!
Looks hard but it's pretty simple, the hardest part is calulating your proper start sector.
Now go boot up your VM and you will find that swap has increased to 6GB and your root volume has an additional 34GB of space in it!

This all happens in less than a minute if you script/automate it all and doesn't have any of the conversion or sparse to non-sparse issues that virt-resize has.

Enjoy!




