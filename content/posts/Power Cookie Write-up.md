---
title: "picoCTF — Power Cookie Write-up"
date: 2026-11-25
draft: false
tags:
  - picoCTF
  - Web Exploitation
  - CTF
  - Cookie Poisoning
categories:
  - Write-ups
ShowToc: true
---

## Overview
- **Platform:** picoGym (picoCTF)
- **Category:** Web Exploitation
- **Difficulty:** Medium
- **Key Concept:** Client-Side Trust / Cookie Poisoning

Upon launching the instance, we are presented with a minimal web application titled **Online Gradebook**. The objective is to bypass the guest restriction and retrieve the flag.

## Reconnaissance
The landing page contains a single button labeled **"Continue as guest"**. Clicking it redirects to `/check.php`, which displays the following message:

> *"We apologize, but we have no guest services at the moment."*

Inspecting the browser's Developer Tools (**Application > Cookies**) reveals a plaintext cookie named `isAdmin` with a value of `0`.

## Vulnerability Analysis
This is a **Broken Access Control** vulnerability caused by trusting client-side state for authorization decisions. 

 `/check.php` appears to read the `isAdmin` cookie directly from the incoming HTTP request and grant administrative access whenever its value is `1`. Because cookies are stored and controlled on the client side, any user can forge or modify them however they want.

## Exploitation
Modifying `isAdmin` to `1` on the landing page before clicking the button did not work, as clicking **"Continue as guest"** resets the cookie back to `0` before redirecting to `/check.php`.

Instead, modifying the cookie on `/check.php` and reloading the page grants admin access and reveals the flag:

My route:
1. Navigate to `/check.php` by clicking **"Continue as guest"**.
2. Open Developer Tools (**Application > Cookies**) and change `isAdmin` from `0` to `1`.
3. Refresh the page to retrieve the flag.

Alternatively, these can be accomplished completely with a single `curl` request:

`curl -b "isAdmin=1" http://<TARGET_HOST>:<PORT>/check.php`

## Remedition

These kind of vulnerabilities appear when the application relies on client-controlled values for authorization. To fix this, use signed tokens and utilize server-side sessions. 
