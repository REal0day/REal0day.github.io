---
layout: post
title: Rekalling the Times I Played with Windows
date: 2017-10-22 21:28:16
description: Tutorial on how to use rekall to dump windows memory
tags: memory hacking rekall forensics windows
categories: blog
---

![Windows Chan](/assets/img/windows-chan.png "Windows Chan"){:class="featured-image"} Windows-Chan

The more and more you learn about Information Security, the more you learn about how insecure things are. From your personal devices that you are to protect, to devices in our environment. I remember reading my CompTIA Sec+ book back when I was a youngling, and remembered a story in there that described a raid where a hacker took a shotgun and shot his hard drive before the Feds grabbed him and separated him from his computer.

---

## Another Reason for RAM Acquisition
![Hawkeye](/assets/img/hawkeye.png "Hawkeye"){:class="featured-image"} Hawkeye

Another reason as to why the Feds separate hackers from their computers is because their RAM holds valuable information so long as the computer is on. That information includes all passwords, websites, and other types or what forensics people call "artifacts."
*A few of the artifacts one can acquire from a memory acquisition:*  
![Benjamin Caudill](/assets/img/BenjaminCaudill.png "Benjamin Caudill"){:class="featured-image"} Benjamin Caudill
(Slide by Benjamin Caudill)

---

## Experimenting with Memory Acquisition

After playing around with some Reverse Engineering challenges, I've been wanting to get deeper and deeper. Down the rabbit hole we go.

### Tools Used:
- [Rekall Memory Forensic Framework](http://www.rekall-forensic.com/)
- [Volatility](http://www.volatilityfoundation.org/)

A few days ago, I wanted to see if I could take an image of my RAM. The test was conducted on my MacBook Air, so I looked for tools to do this. Rekall, created by Google, has a tool called `osxpmem` which will take an image of your RAM. I wrote a script to make this a bit easier. **Root privileges ARE required.** Jonathon Poling goes deeper into how to do this in his terrific [article](http://ponderthebits.com/2017/02/osx-mac-memory-acquisition-and-analysis-using-osxpmem-and-volatility/).

```bash
#!/bin/sh

# Create directories
mkdir /tmp/mem
mkdir /tmp/mem/Memory_Captures

# Takes us to our work env
cp osxpmem_2.0.1.zip /tmp/mem
cd /tmp/mem
unzip osxpmem_2.0.1.zip

# Required to use utilities
sudo chown -R root:wheel osxpmem.app/
sudo osxpmem.app/osxpmem -o Memory_Captures/mem.aff4
sudo osxpmem.app/osxpmem -e /dev/pmem -o Memory_Captures/mem.raw Memory_Captures/mem.aff4

# Unload our kernel extension
sudo osxpmem.app/osxpmem -u
```

I posted my code on GitHub for your convenience and for the commits. No shame in my commit game.
![memory example 1](/assets/img/mem-ex1.png "memory example 1"){:class="featured-image"}

I sent this to Rekall and got some problems with profiles. More learning!

**Profiles are... research**

"A Mac profile includes the structure definitions for the specific kernel version as well as the addresses of important global variables used in analysis."  
— *The Art of Memory Forensics (pg. 784)*

Since Mac profiles are pretty big, they aren't included with all the fun installs of Volatility; however, some are located on their GitHub. Rekall also has profiles as well on their GitHub. This is kinda what it looks like when you don't have the correct profiles.

---

## Switching to Windows for Memory Acquisition
![memory example 2](/assets/img/mem-ex2.png "memory example 2"){:class="featured-image"}
So, I put my Mac image down and picked up the other computer next to me—a family member's Windows 7 computer. Just like I'm sure the rest of you do, I reinstalled the OS for my family because someone who studies Computer Science means I can help remove your malware and spyware you got installed from just doing "normal computer stuff." 😒

I set a password via login and wanted to take an image of a Windows computer to see if I could get the login password from it!

Initially, I was worried and thought taking an image might be difficult, or maybe I'd have to dive pretty deep into a tool to figure out the password. I am very grateful for the heroes and heroines that took on this frontier.

**"If I have seen further than others, it is by standing upon the shoulders of giants."**  
— Isaac Newton

---

## Windows Memory Acquisition

I accomplished this goal with two lines. But first, the image:

1. Get `winpmem` (at the time of writing, `winpmem-2.1.post4.exe`).
2. Open `cmd.exe` as Administrator (Right-Click and click "Run as Administrator").
3. Run the following command:
   ```bash
   winpmem -o Win7Image.aff4
   ```

![ram dump](/assets/img/ram_dump.png "ram dump"){:class="featured-image"}

Got RAM?! YEAH BITCH!

Alright, so now let's open it up in Rekall.
Rekall is very well-documented on how to set up their environment.

```bash
$ virtualenv /tmp/MyEnv

New python executable in /tmp/MyEnv/bin/python
Installing setuptools, pip...done.

$ source /tmp/MyEnv/bin/activate
$ pip install --upgrade setuptools pip wheel
$ pip install rekall-agent rekall
```

If you don't have virtualenv, install it with:
```bash
sudo pip install virtualenv
```

Assuming you got it installed, check it out:

We got it loaded in! Now what are those two lines? Well...loading was one of them.  

Now use the plugin **mimikatz**:

```bash
rekall -f Win7Image.aff4 mimikatz
```

## Profit

If you have any questions, as I wrote this pretty late and ran through it, please feel free to contact me!

The quickest way would be Twitter. Send me more things to play with or correct me if I'm wrong!  

Until next time, hackers!

---

## References

- [DEF CON 21 – Offensive Forensics: CSI for the Bad Guy by Benjamin Caudill](https://www.youtube.com/watch?v=0AwI6YrV2h4)  
- [DEF CON 24 – int0x80 – Anti Forensics AF by DualCore](https://www.youtube.com/watch?v=_fZfDGWpP4U)  
- *Taking Memory Forensics to the Next Level* by Jamie Levy  
- [Rekall Memory Forensics Cheatsheet](https://digital-forensics.sans.org/media/rekall-memory-forensics-cheatsheet.pdf)  
- [OSX (Mac) Memory Acquisition and Analysis Using OSXpmem and Volatility by Jonathon Poling](http://ponderthebits.com/2017/02/osx-mac-memory-acquisition-and-analysis-using-osxpmem-and-volatility/)  
- [Benjamin Caudill's Twitter](https://twitter.com/rhinosecurity)  
- [DualCore's Twitter](https://twitter.com/dualcoremusic?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor)  
- [Taking Memory Forensics to the Next Level Talk by Jamie Levy](https://www.youtube.com/watch?v=GqtAdYS0xyE)  
- [Jamie Levy's Twitter](https://twitter.com/gleeda)  
- [Jonathon Poling's Blog](http://ponderthebits.com/author/jp/)  
