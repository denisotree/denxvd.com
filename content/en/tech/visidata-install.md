---
title: Visidata - Installation and Environment Setup
date: 2024-12-24T14:28:57
description: This article provides a complete guide to installing and setting up Visidata on your computer, along with tips to streamline the process and enhance environment control.
tags:
  - visidata
  - bash
  - linux
  - macos
  - review
categories:
featured_image: /tech/visidata-install/visidata-install.jpg
featured_image_caption: ""
summary: A complete guide to installing and setting up Visidata, along with tips to streamline the process and enhance environment control.
DisableComments: true
draft: false
---

![](/tech/visidata-install/visidata-install.jpg)

**Visidata** is a powerful terminal-based tool for working with tabular data. It allows for quick data analysis, filtering, and transformation with minimal resource usage. Learn more about it [here]({{< relref "visidata" >}}).

Since **Visidata** is a cross-platform application, there are many ways to install it. I recommend installing it as a `pip` package for your primary global Python interpreter. This provides flexibility in managing additional packages and dependencies that Visidata might require for certain use cases.

---

## Installing Visidata on macOS/Linux

### Step 1: Ensure Python is Installed

Visidata is distributed as a Python package, so you'll need Python version 3.7 or higher (preferably later for the latest features). To check your Python version, run:

```bash
python3 --version
```

If Python is not installed, you can install it using your system's package manager:

#### macOS:

Install [Homebrew](https://brew.sh/) (if it's not already installed), then run:

```bash
brew install python
```

#### Linux:

For most Linux distributions, Python can be installed via the default package manager. Here's how for popular distributions:

- **Ubuntu/Debian**:

```bash
sudo apt update  
sudo apt install python3 python3-pip
```

- **Fedora**:

```bash
sudo dnf install python3 python3-pip
```

- **Arch Linux/Manjaro**:

```bash
sudo pacman -S python python-pip
```

- **CentOS/RHEL**:

For CentOS/RHEL 8 and later:

```bash
sudo dnf install python3 python3-pip
```

For older versions like CentOS 7, you may need to enable the EPEL repository:

```bash
sudo yum install epel-release  
sudo yum install python3 python3-pip
```

If your distribution is not listed, consult its official documentation or compile Python from source.

---

### Step 2: Install Visidata via pip

Once Python is installed, you can use `pip` to install Visidata. Run:

```bash
pip install visidata --upgrade
```

This installs the latest version of Visidata.

---

### Step 3: Verify Installation

After installation, confirm that Visidata works correctly by running:

```bash
vd --version
```

If the command is not recognized, it could mean the directory where `pip` installed Visidata is not in your `PATH` environment variable. To fix this:

1. Find the directory where `pip` installed packages. Run:

```bash
which python3
```

2. Ensure the `bin` directory inside this path is added to your `PATH`. For macOS/Linux, it might look like this:

```bash
export PATH="/opt/homebrew/bin/python3/bin:$PATH"
```

3. Add this line to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or another, depending on your shell) to make it persistent. Reload the configuration with:

```bash
source ~/.bashrc  # For bash  
source ~/.zshrc   # For zsh
```

4. Retry the command:

```bash
vd --version
```

If the issue persists, reinstall Visidata with the `--user` flag:

```bash
pip install --user visidata --upgrade
```

---

### Step 4: Install Additional Dependencies

Visidata uses various Python libraries for extended functionality. To enhance its capabilities, install the following:

```bash
pip install PyYAML lxml pandas xlrd openpyxl h5py xport savReaderWriter --upgrade
```

These libraries enable Visidata to handle files like Excel, YAML, HDF5, XML, XPT, SAV, Arrow, and HTML tables.

Additionally, install the `tabulate` library to export data as beautiful tables:

```bash
pip install tabulate --upgrade
```

Supported table formats are detailed in the [tabulate documentation](https://github.com/astanin/python-tabulate).

---

## Installing Visidata on Windows

### Step 1: Ensure Python is Installed

On Windows, you’ll also need Python 3.7 or higher. To check, run in Command Prompt or PowerShell:

```bash
python --version
```

If Python is not installed, download it from [python.org](https://www.python.org/downloads/) and install it. During installation, ensure you check **"Add Python to PATH"** for easier access.

---

### Step 2: Install Visidata via pip

With Python installed, run the following in Command Prompt or PowerShell:

```bash
pip install visidata --upgrade
```

This installs the latest version of Visidata.

---

### Step 3: Verify Installation

Check that Visidata works by running:

```bash
vd --version
```

If the command is unrecognized, it might be because the `Scripts` directory (where pip installs executables) is not in your `PATH`. To fix this:

1. Locate the `Scripts` directory, typically:

```bash
C:\Users\<Your_Username>\AppData\Local\Programs\Python\Python<version>\Scripts\
```

2. Add this path to your `PATH` environment variable:
   - Press **Win + S** and search for "Environment Variables."
   - Under "System Variables," find `Path` and click "Edit."
   - Add the `Scripts` directory path.

3. Restart Command Prompt and check again:

```bash
vd --version
```

If it still doesn't work, reinstall Visidata with the `--user` flag:

```bash
pip install --user visidata --upgrade
```

---

### Step 4: Install Additional Dependencies

Enhance Visidata's functionality by installing additional libraries:

```bash
pip install PyYAML lxml pandas xlrd openpyxl h5py xport savReaderWriter --upgrade
```

Also, install `tabulate` for exporting data in various table formats:

```bash
pip install tabulate --upgrade
```

Refer to the [tabulate documentation](https://github.com/astanin/python-tabulate) for supported formats.

---

Now you're ready to use Visidata for data analysis! For any issues during installation, check the [official documentation](https://www.visidata.org/docs/).
