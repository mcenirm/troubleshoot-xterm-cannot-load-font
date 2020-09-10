Weird error message:

> /usr/bin/xterm: cannot load font '-misc-fixed-medium-r-semicondensed--13-120-75-75-c-60-iso10646-1'

https://unix.stackexchange.com/a/433325

> With Redhat7 (or CentOS7), you need only two packages for bitmap-fonts with xterm. One package (xorg-x11-fonts-misc) covers all but a special case for the menus. Other systems will use different package names (and divide up the fonts in different ways).

I do not have a way to test the error or the fix in the original context (cron task that uses Xvfb on RHEL7), so hooray for Docker!


## More details about the original context

Xvfb starts with the `-fp unix/:7100` option, but there is no font server running on the system. This appears to be a hold-over from at least three generations of systems for this particular setup, which started on SGI IRIX. This explains why installing the new font package made no difference, since its fontpath did not include the expected `catalogue:/etc/X11/fontpath.d` entry. With the missing font server, `xlsfonts` showed only:

```
-misc-fixed-medium-r-semicondensed--0-0-75-75-c-0-iso8859-1
-misc-fixed-medium-r-semicondensed--13-100-100-100-c-60-iso8859-1
-misc-fixed-medium-r-semicondensed--13-120-75-75-c-60-iso8859-1
6x13
cursor
fixed
```

Useful commands:

* `xset q | grep -A1 -i font` - show the active font path
* `xset fp rehash` - reread the fonts details without changing the font path
* `xset fp default` - reset the font path to defaults
* `xset +fp catalogue:/etc/X11/fontpath.d` - prepend the expected catalogue to the active font path


## Docker weirdness

The "centos:7" Docker image never even shows the "cannot load font" error message.

In contrast, the original RHEL7 system and the "bento/centos-7" (v202008.16.0) Vagrant box both show the error message when starting xterm. (see [the list of RPM differences](docker-vs-vagrant.rpm-qa.diff))


## References

[X Logical Font Description](https://wiki.archlinux.org/index.php/X_Logical_Font_Description) from archlinux


## Previous effort

* See what X11 font packages are already there
   ```shell
   rpm -qa '*x11*font*'
   ```

```console
[redacted@redacted ~]$ rpm -qa '*x11*font*'
xorg-x11-font-utils-7.5-21.el7.x86_64
xorg-x11-fonts-Type1-7.5-9.el7.noarch
[redacted@redacted ~]$
```

* Install recommended package
   ```shell
   sudo yum install xorg-x11-fonts-misc
   ```

```console
[redacted@redacted ~]$ sudo yum install xorg-x11-fonts-misc
…
Installed:
  xorg-x11-fonts-misc.noarch 0:7.5-9.el7

Complete!
[redacted@redacted ~]$
```

* Show other X11 fonts
   ```shell
   sudo yum list xorg-x11-fonts\*
   ```

```console
[redacted@redacted ~]$ sudo yum list xorg-x11-fonts\*
…
Installed Packages
xorg-x11-fonts-Type1.noarch                           7.5-9.el7               @anaconda/7.3
xorg-x11-fonts-misc.noarch                            7.5-9.el7               @_local
Available Packages
xorg-x11-fonts-100dpi.noarch                          7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-75dpi.noarch                           7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-1-100dpi.noarch                7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-1-75dpi.noarch                 7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-14-100dpi.noarch               7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-14-75dpi.noarch                7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-15-100dpi.noarch               7.5-9.el7               rhel-7-server-optional-rpms
xorg-x11-fonts-ISO8859-15-75dpi.noarch                7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-2-100dpi.noarch                7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-2-75dpi.noarch                 7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-9-100dpi.noarch                7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ISO8859-9-75dpi.noarch                 7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-cyrillic.noarch                        7.5-9.el7               rhel-7-server-rpms
xorg-x11-fonts-ethiopic.noarch                        7.5-9.el7               rhel-7-server-rpms
[redacted@redacted ~]$
```

* Does any package provide an ISO10646 font?
   ```shell
   sudo yum provides '/usr/share/X11/fonts/*ISO10646*'
   ```

```console
[redacted@redacted ~]$ sudo yum provides '/usr/share/X11/fonts/*ISO10646*'
…
No matches found
[redacted@redacted ~]$
```

Apparently, [ISO 10646 is a successor to ISO 8859](https://en.wikipedia.org/wiki/Universal_Coded_Character_Set).

* Install guess package
   ```shell
   sudo yum install xorg-x11-fonts-75dpi xorg-x11-fonts-ISO8859-1-75dpi
   ```

```console
[redacted@redacted ~]$ sudo yum install xorg-x11-fonts-75dpi xorg-x11-fonts-ISO8859-1-75dpi
…
Installed:
  xorg-x11-fonts-75dpi.noarch 0:7.5-9.el7        xorg-x11-fonts-ISO8859-1-75dpi.noarch 0:7.5-9.el7

Complete!
[redacted@redacted ~]$
```

TODO: Find reference for why I learned to look in fonts.dir.

* Look for fonts.dir files
   ```shell
   rpm -qal | grep -i fonts.dir | xargs -t -n1 rpm -qf
   ```

```console
[redacted@redacted ~]$ rpm -qal | grep -i fonts.dir | xargs -t -n1 rpm -qf
rpm -qf /usr/share/X11/fonts/75dpi/fonts.dir
xorg-x11-fonts-ISO8859-1-75dpi-7.5-9.el7.noarch
xorg-x11-fonts-75dpi-7.5-9.el7.noarch
rpm -qf /usr/share/fonts/default/ghostscript/fonts.dir
ghostscript-fonts-5.50-32.el7.noarch
rpm -qf /usr/share/X11/fonts/misc/fonts.dir
xorg-x11-fonts-misc-7.5-9.el7.noarch
rpm -qf /usr/share/X11/fonts/Type1/fonts.dir
xorg-x11-fonts-Type1-7.5-9.el7.noarch
rpm -qf /usr/share/fonts/liberation/fonts.dir
liberation-fonts-common-1.07.2-16.el7.noarch
rpm -qf /usr/share/X11/fonts/75dpi/fonts.dir
xorg-x11-fonts-ISO8859-1-75dpi-7.5-9.el7.noarch
xorg-x11-fonts-75dpi-7.5-9.el7.noarch
[redacted@redacted ~]$
```

* Look in fonts.dir files for the missing font
   ```shell
   rpm -qal | grep -i fonts.dir | xargs grep -e -misc-fixed-medium-r-semicondensed--13-120-75-75-c-60-iso10646-1
   ```

```console
[redacted@redacted ~]$ rpm -qal | grep -i fonts.dir | xargs grep -e -misc-fixed-medium-r-semicondensed--13-120-75-75-c-60-iso10646-1
/usr/share/X11/fonts/misc/fonts.dir:6x13.pcf.gz -misc-fixed-medium-r-semicondensed--13-120-75-75-c-60-iso10646-1
[redacted@redacted ~]$
```

* Which package owns that file?
   ```shell
   rpm -qf /usr/share/X11/fonts/misc/6x13.pcf.gz
   ```

```console
[redacted@redacted ~]$ rpm -qf /usr/share/X11/fonts/misc/6x13.pcf.gz
xorg-x11-fonts-misc-7.5-9.el7.noarch
[redacted@redacted ~]$
```

That is, in theory, the only required package was the recommended one, "xorg-x11-fonts-misc".
