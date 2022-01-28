---
title: Set up gdb in macOS
date: 2022-01-28 20:31:29
tags: gdb
categories: C 
---

# How to setup gdb in macOS
[referencelink-zhihu](https://zhuanlan.zhihu.com/p/68398728)

<!--more-->

## Step1: install gdb
```sh
brew install gdb
```

The gdb executable is in /usr/local/bin/gdb

**GDB is not plug-and-play in macOS, it needs setup since macOS does not allow debugger to debug codes. macOS does not want its kernel be debugged**

## Step2: setup certificate

* `cmd + space` open spotline, and type `keychain access`

* In upper-left menu, select `certificate assistant -> create a certificate`
* name it `gdb-cert` or what ever you like
  * Identity Type: Self Signed Root
  * Certificate Type: Code Signing
* Continue ahead, until `position of certificate (select system)`
* double-click the certificate, `Trust -> When using this certificate -> Always Trust`

## Step3: in terminal

* create a file named `gdb-entitlement.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.cs.debugger</key>
    <true/>
</dict>
</plist>
```

* open the terminal
  * `codesign --entitlements gdb-entitlement.xml -fs gdb-cert /usr/loacl/bin/gdb`
  * `echo "set startup-with-shell off" >> ~/.gdbinit`

**You are all set! Enjoy gdb!**