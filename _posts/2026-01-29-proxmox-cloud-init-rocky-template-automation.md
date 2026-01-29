---
layout: post
title: "Proxmox, Cloud-init, and Automating the Rocky 9 Template"
date: 2026-01-29
categories:
  - infrastructure
  - Linux
  - ProLux
  - Proxmox
---

**Date:** January 29, 2026

---

## What I'd Been Ignoring

I'd been using Proxmox for a while without really paying attention to Cloud-init. It was one of those things that lived in the UI and the docs: "enable Cloud-init when creating a VM," "use this for provisioning." I'd clone a template, spin up a Rocky 9.5 VM, and move on. I hadn't dug into how the template was built or how to change it in a repeatable way.

Yesterday I did just that. 

---

## The Need

I use a Rocky 9.5 template for new servers. The idea was to specify in Ansible the request for just creating a new server based on a template and having all the networking set up during the spawn process, or the provisioning process. This wasn't really possible without bringing the server up, changing the network settings, and then rebooting. The other option is to use Cloud-init, which allows you to provision this system with settings based on code.

But before we could do that, I had to modify the base template. This required a few additional steps:

1. Turn the template back into a regular VM (or clone it to a VM).
2. Boot it, add the packages, clean up.
3. Convert it back into a template.

I wanted to do that in a way I could repeat, not a one-off click-through. So I had to learn how the template was structured and how Cloud-init fit in.

---

## What I Learned

**Cloud-init in Proxmox.** Cloud-init runs early in the boot process and handles things like hostname, network, SSH keys, and user data. In Proxmox you typically enable it when creating a VM from an image or when converting a VM to a template. The template then ships with Cloud-init configured so that when you clone from it, you can pass in a new hostname, IP, and so on without ever logging in. I hadn't really used that flow before; I'd just been cloning and then configuring by hand. Looking at the template and the Cloud-init options made it clear how the "clone and customize" workflow is supposed to work.

**Editing the template.** I had to "open" the template: in practice that meant cloning it to a VM, starting the VM, installing the packages I needed, doing any cleanup (e.g. clearing logs, trimming history), then shutting it down and converting it back to a template. That way the next clone gets the new package set without me having to install those packages on every new server.

**Automating it.** I didn't want to do that dance manually every time I need to change the base image. So I turned the steps into something scriptable: clone template to VM, start VM, wait for SSH (or use Proxmox guest agent), run the package install and cleanup, shut down, convert back to template. That way updating the Rocky 9.5 template is a single run instead of a bunch of clicking and waiting.

---

## Why It Mattered

Before this I was treating the template as a black box. Now I have a clear picture of how it's built and how to change it. Cloud-init went from "something Proxmox mentions" to "the mechanism that makes clone-to-new-server actually work." And having an automated path from "template" to "editable VM" to "updated template" means I can refresh the base image whenever I need to without ad hoc manual steps.

---

## Takeaway

Sometimes you don't learn a thing until you need it. I'd ignored Cloud-init in Proxmox until I had to modify the Rocky 9.5 template and bake in new packages. Doing that once by hand was enough to see the pattern; automating it means the next time I need to change the template, I run a script instead of repeating the process step by step.

#Linux #ProLug

