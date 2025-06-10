---
layout: post
title: Fix CERTIFICATE_VERIFY_FAILED Issue For Python 3.13 on macOS
key: 20250610
tags: Python SSL certifi
s: 2506101218
---

# Fix CERTIFICATE_VERIFY_FAILED Issue For Python 3.13 on macOS

When using Python 3.13 on macOS, you may encounter a `CERTIFICATE_VERIFY_FAILED` error when using `pip install`, which uses HTTPS requests.

The error message looks like this:

```
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1028)'))': /simple/certifi/
```

Although the official Python documentation suggests running the `Install Certificates.command` script after Python is installed, you will still get the same error when you run the script.

## Solution

First, you need to install the `certifi` package manually with the `--trusted-host` option to bypass the SSL verification:

```bash
# Use pip or pip3
pip install certifi --trusted-host pypi.org --trusted-host files.pythonhosted.org
```

Then, set the `SSL_CERT_FILE` environment variable to point to the `certifi` certificate bundle:

```bash
export SSL_CERT_FILE=$(python -m certifi)
```

After this variable is set, you should be able to run all the `pip install` commands without the `CERTIFICATE_VERIFY_FAILED` error.

If you don't want type the command every time you use Python scripts, you can add it to the shell profile, like `.bash_profile`, `.zshrc`, or `.bashrc` depending on your shell:

```bash
# For zsh users
echo 'export SSL_CERT_FILE=$(python -m certifi)' >> ~/.zshrc
source ~/.zshrc
```
