---
title: "Learning Nix"
date: 2023-12-07T20:38:34-05:00
draft: false
---

# Introduction

I recently bought a new computer for personal use. Like any new computer, I had to set up my favorite software on it: Spotify, Firefox, the One True Editor, and so on.

A reasonable person would go about this problem the way they'd go about most problems: by solving it. For better or for worse, I am not such a person. I decided to automate this setup so that in theory I'd never have to do it again.

As the saying goes, now I've got **two problems.**

This post is a mostly unstructured catalog of my thoughts about Nix. If you a want a more detailed, thorough look at Nix, [Ian Henry's tour of the Nix manual](https://ianthehenry.com/posts/how-to-learn-nix/) is where it's at.

## Before Nix

In a past life, I worked on a system for provisioning MapReduce clusters.
People doing data stuff would click buttons in a UI, go get a cup of coffee, and come back with a ready-to-use cluster for data stuff.

These clusters were often too large (in general, they were just too large -- but that's an aside) to manage by hand, so my team used [Ansible.](https://www.ansible.com/)

[Ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) allow you to write a YAML file that does normal sysadmin stuff like
installing software, configuring software, pipe `curl` into `sh`, and so on. Ansible is a fine tool, but I didn't enjoy using it much personally.

I thought to myself, "Surely I can use something simpler than Ansible, like my own Bash script."

Then I got into about 100 lines of Bash, decided some logic was better handled in a Python script, wrote about 100 lines of Python, and thought again -- surely there's something simpler that's a better fit here.

So obviously I landed on Nix.

## Getting Started

I don't remember when or where I first heard about Nix, but I remember hearing that it was confusing and poorly documented. But that was a long time ago. I'm a smarter person now, and Nix has probably gotten better since then.

I started using Nix by treating it like a black box, which is a strategy that's treated me well for being productive quickly.

I found an [example Nix configuration](https://github.com/dustinlyons/nixos-config/) (thanks Dustin!) that checked some boxes I knew I'd care about:

1. macOS compatibility. This is important, because my new computer is a MacBook.
1. NixOS compatibility. This is a nice-to-have. At the time I wasn't sure if I'd like using Nix, but if I did, I would want to use NixOS for dev/homelab hosts.
1. Nix Flakes and Home Manager. I knew enough about Nix to know that these are "experimental" features that are highly encouraged for new Nix users, so I might as well start with them and avoid having to "unlearn" the less popular methods. I can always go back and learn the "old" ways of doing things later.
1. Secrets. I don't plan to open source my Nix configuration, but if I do, it's good to know I can manage secrets separately.

But most importantly, a really obvious iteration loop: change some files, run `bin/build`, and see if it worked.

The `nix` command has a lot going on:

```sh
$ nix --help | wc -l
897
# As of...
$ nix --version
nix (Nix) 2.18.1
```

And I bet that makes a lot of sense for someone out there, but as a brand new user, not for me. A suite of helper scripts would probably be the first thing I write, so it makes sense to bootstrap it by lifting someone else's repo.

This reduces a daunting space of problems, learning Nix, to a normal process for working on code, which I do every day. Great!

## Arguably Interesting Problems

I won't describe every problem I ran into, because most of them are better described elsewhere or just typical newbie problems.

### A Minor Inconvenience with Doom (Emacs)

tl;dr I fixed a minor inconvenience (extra icons in my Dock) in using Doom Emacs on macOS by also using Chemacs, another Emacs package. Since they're all managed by plain-text config files, and Nix is very amicable to such config management, I was able to get everything working "just right."

One sort of interesting problem I ran into was with getting [Doom Emacs](https://github.com/doomemacs/doomemacs) to work in a Nix-y way.

Emacs in general is pretty Nix-friendly. Most Emacs config is manage declaratively (or at least as plain-text source files) these days, and there's an `emacsPackages` package set that works out-of-the-box with Nix if you'd rather use that.

But Doom Emacs is a little different, in that its installation process is meant for any Emacs user and is generally Nix-agnostic. There's a [community-managed derivation for it](https://github.com/nix-community/nix-doom-emacs), but it's effectively unmaintained, so I didn't bother with it.

Luckily, the creator of Doom Emacs is [also a Nix user!](https://github.com/hlissner/dotfiles) Using his configuration I was able to get a working Doom Emacs installation, with one small problem.

I'm using macOS, which provides a nice-looking Dock to display my favorite and currently-running applications. When I'm running `Emacs.app` (installed via Nix), everything looks fine. I've got one icon for Emacs, and nothing else, because I'm not running anything else. But this gives me Emacs, not Doom Emacs, because the config lives in different places: `~/.emacs.d` vs. `~/.doom.d` and `~/.config/emacs`.

But when I run `doom run`, I get Doom Emacs, but it uses a separate icon in the Dock than regular `Emacs.app`. That's not terrible, but:

1. I can't easily call `doom run` from the dock
1. The extra icon is ugly, and surely I didn't pay the Apple Tax to live with minor eyesores like that

So I tried setting up an [Automator script](<https://en.wikipedia.org/wiki/Automator_(macOS)>) as an application to run `doom run` so I could run it from the dock, but this still gave me extra icons. In fact, it gave me an extra one, which pointed to the Automator script. That icon pointing to the Automator was also a default "text file" icon, which was even uglier.

I needed some way to trick my `Emacs.app` into running Doom's config as-is -- not as a separate executable.

Enter [Chemacs](https://github.com/plexus/chemacs2). Chemacs is an "Emacs profile switcher," or a "bootloader for Emacs,"" which lets you easily switch between different Emacs configs by basically adding one more dotfile to the mix.

So now my Nix config looks pretty simple:

```nix
# Wherever you'd define your dotfiles, or however else you define your emacs config:
{ pkgs, config, ... }:

{
  ".emacs-profiles.el" = {
    text = ''
      (("default" . ((user-emacs-directory . "${XDG_CONFIG_HOME}/emacs")))
       ;; Any other profiles you care about go here.
       )
    '';
  };
}
```

That's it. Get Doom and Chemacs playing nicely and you remove the minor inconvenience of extra icons in your Dock.

## Do I Recommend Nix?

So far:

1. Nix the package manager, yes.
1. Nix the operating system, it's too early in my usage to tell, sorry.

Unless:

1. You don't like side quests. If you're more about the destination than the journey, you might as well skip Nix, because you'll sink a lot of time into the "extra stuff" rather than what you set out to do originally. I was already an Emacs user so this didn't bother me.
1. You don't like figuring things out.
1. You manage fewer than 3 hosts.

I absolutely _hate_ the inconsistency between hosts that I was managing: my work laptop, my development VM at work, my personal laptop, and a homelab server. I dislike it enough that I figured it's worth a weekend to grapple with Nix. I think that's paid off pretty well already, and will continue to do so as long as I maintain my discipline about it.

## References

1. https://ianthehenry.com/posts/how-to-learn-nix/
1. https://www.joseferben.com/posts/installing_only_certain_packages_from_an_unstable_nixos_channel/
1. https://xeiaso.net/talks/asg-2023-nixos/

I stand on the shoulders of giants:

1. The repo I started mine with: https://github.com/dustinlyons/nixos-config/
1. dotfiles from hlissner, the creator of Doom Emacs https://github.com/hlissner/dotfiles
