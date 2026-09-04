# Pu-239: atomic, unbreakable, lightning fast.

## How Pu-239 Works

Pu-239 combines two ideas: a sealed, atomically updated root filesystem inspired by SteamOS and Fedora Silverblue combined with a performance kernel, linux-cachyos, built from source and hosted on my own repository. This is the first distro where you don't have to make any compromise between ease of use + safety and bleeding-edge + performance optimizations.

### Atomic deployments with ostree

Your OS is not one mutable directory tree, but a series of deployments managed by ostree. Think of it like git for your operating system.

* Every update creates a complete new root tree and leaves the old one untouched.
* Switching between them is atomic, so you are always either fully on the old tree or fully on the new one. There is no half updated state.
* Unchanged files are hardlinked between deployments, so keeping a rollback copy costs almost no extra disk space.

```text
/boot/                             EFI system partition (bootloader, kernel, initramfs)
/ostree/repo/                      ostree object store (shared files)
/ostree/deploy/pu-239/
  deploy/<hash>.0/                 deployment A (sealed read only at boot)
  deploy/<hash>.1/                 deployment B (sealed read only at boot)
  var/                             shared /var (writable, survives everything)
    home/                          your files
    lib/flatpak/                   installed Flatpaks
    lib/overlays/etc/              persistent /etc modifications
```

### The root is sealed

At boot, the active deployment is mounted read only. You cannot install packages onto the root with pacman. This is what makes updates safe, because nothing you run can corrupt the OS tree.

* System updates replace the entire tree atomically.
* User applications are installed as Flatpaks, and Flathub is preconfigured.
* Your configuration still works normally, thanks to an overlay.

### Updates cannot brick the system

Every deployment gets a boot entry with a tries counter, handled natively by systemd-boot, no custom scripts.

```text
pu-239-<hash>+3.conf   (3 boot attempts remaining)
pu-239-<hash>.conf     (blessed, booted successfully at least once)
pu-239-<hash>+0.conf   (exhausted, skipped by the bootloader)
```

After an update, the new deployment boots with +3. If it fails to reach a successful boot, the counter is decremented on the next attempt. Once exhausted, the bootloader skips it automatically and boots the previous good deployment. A broken update reverts itself on reboot, so you never have to fix anything manually. With LUKS encryption enabled, the fallback boot asks for your passphrase like any other boot, since the disk is unlocked fresh on every start.

### Your data is never part of a rollback

The /var directory, including your home directory, Flatpak installs, and /etc modifications, lives outside the deployments and is shared by all of them. Rolling back the OS never touches your files.

The /etc directory deserves special mention. Each deployment ships a read only default /etc, over which Pu-239 mounts an overlay filesystem whose writable layer lives in /var. Your edits, like Wi-Fi passwords and the hostname, persist across updates and rollbacks, while untouched config files still receive upstream updates as usual.

### The kernel

Pu-239 ships linux-cachyos, the performance kernel of the CachyOS project, featuring a 1000 Hz tick rate, EEVDF and BORE scheduling, and optimizations tailored to modern CPUs. We compile it ourselves from their GPL licensed build scripts and host it on our own infrastructure, in accordance with CachyOS repository policy. See the CREDITS file for details.

### Stock Arch-Linux underneath

The installer does not flash a giant prebuilt image. It downloads packages from the live official Arch mirrors and builds the first deployment on your machine during installation, just like a normal Arch install, except atomically versioned from the very first boot.

Day to day, updating looks exactly like regular arch-update. The only difference is that the apply step does not modify your running system. It builds a new deployment, writes a new boot entry, and asks you to reboot. If anything goes wrong after the reboot, the fallback mechanism takes over.

## Timeline

August 26th: Came up with the idea of making Arch Linux atomic like SteamOS combined with the speed and ease of use of CachyOS

August 28th: Finalized reverse-engineering the way SteamOS and Fedora Silverblue make the distro atomic, finalized the overall planned design of the distro, started coding

August 30th: Setteled on the name "Pu-239" as "Arch-Atomic" would not have been allowed and weapon-grade Plutonium-239 as a reference to atomic sounded funny. "Plutonium" was too basic, "PlutoniumOS" sounded too much like a GrapheneOS copy, "Plutonium-239" is too long.

August 31th: First working ISO, wrote the basic code with no focus on visual appeal or QOL, verified functionality of the installer and the atomic filesystem

September 1st: Registered pu-239.org with Hetzner and introduced some ugly but hand-made branding

September 4th: I took a break from working on the OS and got this site online. 0.0.1 will likely be released some time next week, I'm only working on the quality of life stuff at this point.

### Roadmap of what still needs to be done for 0.0.1 to be released

- Set up the repo for pacman at repo.pu-239.org and push all the kernels there.

- Add automatic hardware detection, the current ISO just installs every single CPU and GPU driver even tough I only use Intel hardware.

- Make everything look prettier, I'm no front-end expert as you might be able to tell from this site.

### Version numbers

The 0.0.X versions are pre-release versions meant for people interested in this project to install on non-critical devices. I can NOT guarantee these versions to be stable and viable to daily-drive.

For 0.1.0 to be released: have everything looking clean and working stable, ability to be a daily-driver verified by a small but meaningful amount of users.

For 1.0.0 to be released: to have it stable, verified, tested, liked and recommended by a wide community plus compile and ship every single package optimized for x86_64, x86_64-v3, x86_64-v4 and x86_64-znver4.

### Current hardware and costs

Intel Core Ultra 7 270K Plus and 32GB 7200Mhz DDR5 to compile the linux-cachyos kernels in x86_64, x86_64-v3, x86_64-v4 and x86_64-znver4 - dontletmywifeknowwhatthispccostintotal.99€ yearly

Hetzner Webhosting L + .org domain to host the website plus the four repositories - 136.65€ yearly

Planned: buy at least a Hetzner EX44 server to be able to ship every single package optimized for x86_64, x86_64-v3, x86_64-v4 and x86_64-znver4 - 842.52€ yearly

## Disclaimers

This is a distro based on Arch Linux with an atomic design inspired by SteamOS and Fedora Silverblue plus optimizations inspired by CachyOS. However it is in no way affilliated with any of the mentioned distros, nor does it try to discredit them.

Arch Linux is an amazing base to build a distro on. The flexibility, customization and bleeding edge updates are unmatched. Valve through their work on SteamOS, Proton and contributions to other projects like KDE Plasma or Wine has been a driving force in making Linux something actually usable to daily-drive. The CachyOS team is doing amazing work to improve the speed and ease of use of Arch Linux with their kernel optimizations, CPU-architecture specific packages and repos. I personally still use CachyOS on my main workstation.

This project minimizes and heavily limits the use of generative artificial intelligence, contrary to other certain distros. Transformer models for media generation will NEVER be used, no exeption. I'd rather have an ugly logo that I did myself in GIMP than ask an AI to do it. Large language models have never and will never be used for: coming up with ideas, making decisions, vibe-coding user-facing and critical code, writing documentation like this, interacting with users and so on. My stance on AI in general is that it is super useful, but in 95% of cases it's used for all the wrong things and reasons, in unethical and enviornmentally harming conditions. No proprietary models running in data-centers are ever used, instead I exclusively run open-source large language models on local hardware, currently Qwen3.8-27B on my Intel Arc Pro B70 GPU. This local large language model is exclusively used to automate non-critical, non-user-facing manual tasks like for example writing an automated script for building the CachyOS kernel locally on my workstation and pushing it to the repository or for translations to languages I don't speak and don't have access to a native-speaking translator as an AI-generated translation is better than not having one at all.

I don't collect, store or sell any information about you or your usage, neither on the website, nor in the OS itself. I don't even know how.
