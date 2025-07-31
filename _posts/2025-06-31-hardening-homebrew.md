---
title: "Hardening configuration of Homebrew on MacOS"
image: "../assets/img/2025-06-31-hardening-homebrew.png"
categories: [MacOS]
tags: ["macos", "homebrew", "security"]
---
Installing Homebrew is always one of the very first things I do when setting up a clean install of MacOS. Which I basically do _at least_ once a year, when a new version of MacOS is released. I would not have been able to use my Mac without the tools provided through Homebrew. It's absolutely essential!

Installation is a straight forward procedure. You just take a script from the Internet, download and run it in the terminal. During this process you provide this script, known as the Homebrew installer, with yourpassword to give it sudo access to control your computer.

``` bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

_Someone_ knows what this executes, most don't. It does not help that it is without any means of verifying authenticity or integrity.

As crazy as it sounds, I do this, and it is actually the recommended way to install Homebrew. This smells bad, and is why I opted to review and change the default configuration of Homebrew to be a bit more secure.

## Hardening the configuration

Homebrew checks for a <a href="https://github.com/orgs/Homebrew/discussions/5232#discussioncomment-8919542" target="_blank" rel="noreferrer">systemwide configuration file</a>, located at `/etc/homebrew/brew.env`. This file is not created by the installation script, and must be created manually. I prefer this method over environment variables in `.zshrc` or `.bashrc` as it ensures correct configuration regardless of what shell I use, or how Homebrew is invoked.

```bash
sudo mkdir /etc/homebrew/
sudo vi /etc/homebrew/brew.env

HOMEBREW_SYSTEM_ENV_TAKES_PRIORITY=1
HOMEBREW_NO_ANALYTICS=1
HOMEBREW_NO_INSECURE_REDIRECT=1
HOMEBREW_CASK_OPTS_REQUIRE_SHA=1
```

## System environment takes priority

The first line `HOMEBREW_SYSTEM_ENV_TAKES_PRIORITY=1` tells Homebrew to change it's default prioritization of configuration. System environment files now take priority over all other environment variables, and make it harder to override my configuration.

## Disable analytics and telemetry

The second line `HOMEBREW_NO_ANALYTICS=1` disables the analytics and telemetry that Homebrew collects by default. Fun fact: The Homebrew installer temporarily disables telemetry collection during the installation by use of environment variable HOMEBREW_NO_ANALYTICS_THIS_RUN.

Data sent is "anonymized", and without personal identifiable information. But I still don't like it. I don't necessarily trust:

- That they uphold their promise of anonymization.
- The security of the service they use to store the data.
- That others don't find a way to deanonymize it.
- That no one uses it as a pattern for deep packet inspection.

Running this command should also disable it permanently:

```bash
$ brew analytics off
```

Just for safe measure, I do both...

You can read more about the data collected by Homebrew <a href="https://docs.brew.sh/Analytics" target="_blank" rel="noreferrer">here</a>.

## Disable insecure redirects

By default Homebrew allows insecure redirects, which means that it will follow redirects from encrypted HTTPS to plaintext HTTP. This is a security risk, as it allows an actor on the network between your machine and the source destination to intercept your download.

Setting `HOMEBREW_NO_INSECURE_REDIRECT=1` disables this behavior, and make it harder for an attacker to intercept or monitor the traffic.

## Require SHA checking for Casks

SHA checking is a way to verify the integrity of files downloaded. Homebrew does not (<a href="https://github.com/drduh/macOS-Security-and-Privacy-Guide/commit/62c84014ebec52abe37768cee9247e57c19743cd" target="_blank" rel="noreferrer">did not?</a>) require SHA checking for Casks. I'm actually not sure if this is still the default behavior. Regardless, I want it to be difficult to disable SHA checking of Casks. And that is why `HOMEBREW_CASK_OPTS_REQUIRE_SHA=1` is set.

### Thank you for reading

Do you have any special configuration you recommend, questions or improvements? [Please reach out](/contact).
