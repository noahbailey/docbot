---
title: Laptop Setup
description: 
published: true
date: 2019-07-20T19:48:58.723Z
tags: 
---

## Laptop power management

tlp is a set and forget power management system with sane defaults. 

    sudo pacman -S tlp

Enable the service: 

    sudo systemctl enable tlp
    sudo systemctl start tlp
    
### Tuning Power Management

To get a basic assessment of the system, run: 

    sudo tlp-stat
    
It will give basic tips for improving power management. 
