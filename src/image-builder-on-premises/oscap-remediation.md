# OpenSCAP Remediation

`osbuild-composer` now provides the ability to build security hardened images using the [OpenSCAP] tool. 
This feature is available for `RHEL 8.7` (& above) and `RHEL 9.1` (& above).

[OpenSCAP]: https://github.com/OpenSCAP/openscap/blob/maint-1.3/docs/manual/manual.adoc

## OpenSCAP

The `OpenSCAP` tool enables users to scan images for vulnerabilities and then remediate the non-compliances according to
predefined security standards. A limitation of this is that it is not always trivial to fix all issues after the first
boot of the image.

## Build-time Remediation

To solve this issue, an [osbuild stage] runs the `OpenSCAP` tool on the filesystem tree while the image is being built. The `OpenSCAP` tool runs 
the standard evaluation for the given profile and applies the remediations to the image. This process enables the user to build a more completely
hardened image compared to running the remediation on a live system.

[osbuild stage]: 'https://github.com/osbuild/osbuild/blob/main/stages/org.osbuild.oscap.remediation'

## Openscap Example
```
[customizations.openscap]
profile_id = "xccdf_org.ssgproject.content_profile_standard"
datastream = "/usr/share/xml/scap/ssg/content/ssg-fedora-ds.xml"
```

`osbuild-composer` exposes to fields for the user to customize in the image blueprints: 

  1) The path to the `datastream` instructions (most likely in the `/usr/share/xml/scap/ssg/content/` directory)
  2) The `profile_id` for the desired security standard
  3) Install openscap via this command: `dnf install scap-security-guide`
  4) Use the command `oscap info /usr/share/xml/scap/ssg/content/<security_profile>.xml` to obtain more information such as the profile id to use
  5) The `profile_id` field accepts both the long and short forms, i.e. `cis` or `xccdf_org.ssgproject.content_profile_cis`.






See the below table for supported profiles.

`osbuild-composer` will then generate the necessary configurations for the `osbuild` stage based on the user
customizations. Additionally, two packages will be added to the image, `openscap-scanner` (the `OpenSCAP` tool)
& `scap-security-guide` (this package contains the remediation instructions).

> :warning: **Note**
The remediation stage assumes that the
`scap-security-guide` will be used for the datastream. This package is installed on the image by default. If another datastream is desired, add the necessary package to the blueprint and specify the path to the datastream in the oscap config.

## Supported profiles

The supported profiles are distro specific, see below:

|                             | Fedora | RHEL 8.7^ | CS9/RHEL 9.1^ |
|-----------------------------|--------|-----------|---------------|
| ANSSI-BP-028 (enhanced)     |        |     x     |       x       |
| ANSSI-BP-028 (high)         |        |     x     |       x       |
| ANSSI-BP-028 (intermediary) |        |     x     |       x       |
| ANSSI-BP-028 (minimal)      |        |     x     |       x       |
| CIS Level 2 - Server        |        |     x     |       x       |
| CIS Level 1 - Server        |        |     x     |       x       |
| CIS Level 1 - Workstation   |        |     x     |       x       |
| CIS Level 2 - Workstation   |        |     x     |       x       |
| CUI                         |        |     x     |       x       |
| Essential Eight             |        |     x     |       x       |
| HIPAA                       |        |     x     |       x       |
| ISM Official                |        |     x     |       x       |
| OSPP                        |    x   |     x     |       x       |
| PCI-DSS                     |    x   |     x     |       x       |
| Standard                    |    x   |           |               |
| DISA STIG                   |        |     x     |       x       |
| DISA STIG with GUI          |        |     x     |       x       |
