---
layout: post
title: "Mixed DPI with X-Server"
date: 2024-01-02 20:30:15 +0100
categories: "linux"
---
Like many of you, I'm forced to use X11 due to the lack of Wayland support for Nvidia GPUs. This wasn't even an issue until I
thought about buying a 4k-Monitor in addition to my 2 1080p-Monitors. To my surprise, my Desktop Environment KDE-Plasma handled
the mixed DPI setup after playing a bit with the Settings pretty well. The real problem came after I changed from KDE-Plasma
to i3-wm because it doesn't handle the Monitors so I have to do it on my own. I played a bit with `xrandr` and the scaling
options but every time I was confronted with the same problem of a blurry image either on the 1080p/4k-Monitor so after googling 
and playing around I found a solution that suited me pretty well.

The idea is to resize the virtual resolution of the 1080p-Monitors to 4k and then scale the image back down to 1080p. 

### Guide
Lets assume we have the following setup:
- Monitor-1 (1080p, left, DP-1)
- Monitor-2 (2160p, middle, DP-2)
- Monitor-3 (1080p, right, DP-3)
<div align="center" style="padding-bottom: 5px;">
  <img src="/assets/2024-01-02/first_setup.png">
</div>
Like I said we now want to increase the virtual resolution of the 1080p-Monitors to 4k **(with the factor of 2)** and in addition 
I also increased the resolution of my 4k-Monitor **(with the factor of 1.5)**. The new resolutions should now look like:
- Monitor-1 (height=2160,width=3840)
- Monitor-2 (height=3240,width=5760)
- Monitor-3 (height=2160,width=3840)
<div align="center" style="padding-bottom: 5px;">
  <img src="/assets/2024-01-02/scaled_setup.png">
</div>
Now we need to calculate the framebuffer dimension this fairly simpley we just sum the scaled monitor widths and for the height we
are using the scaled height with the highest value so in our case: framebuffer (height=3240,width=13440). Before we now write a 
script we have to determine the names of the Monitors which xrandrs assigns them. We can do that by entering the following command
in the terminal:
```Bash
xrandr | grep -w connected
```
For simplicity we will use the same monitor names as we declared above.
{% highlight ruby %}
#!/bin/bash

xrandr --fb 13440x3240 \
  --output DP-1 --scale 2x2 --pos 0x0 \
  --output DP-2 --scale 1.5x1.5 --pos 3840x0 --primary \
  --output DP-3 --scale 2x2 --pos 9600 \
{% endhighlight %}
- `--fb`: Sets the framebuffer to our calculated resolution.
- `--output`: Tpecifies the monitor by its name.
- `--scale`: The scale which will be applied to the monitor.
- `--pos`: The position of the monitor, **make sure it's the scaled resolution**.
- `--primary`: Makes the specified monitor the primary one.

At the end technical it should look similar to this graphic.
<div align="center">
  <img src="/assets/2024-01-02/framebuffer.png">
</div>