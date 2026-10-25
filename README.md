# php74-el9-rpm

Experimental port of the **AlmaLinux 8 PHP 7.4.33-8 RPM source package** to **AlmaLinux 9 (EL9)**.

This project adapts the distribution-maintained PHP 7.4 package for building on EL9. It is **not** a rebuild of an unpatched upstream PHP 7.4.33 release. The starting point is the AlmaLinux 8 `php:7.4` package release `7.4.33-8`, which includes distribution security backports. The EL9 package is being built as `php-7.4.33-8.el9`.

> [!WARNING]
> **Unofficial, experimental, and unsupported.** PHP 7.4 reached upstream end of life in November 2022, but the `php:7.4` stream on AlmaLinux 8 continues to receive distribution maintenance. This independent EL9 port is **not an official AlmaLinux or Red Hat package**, and those vendors do not maintain or support it. Use it at your own risk, especially in production.

## Why this repository exists

AlmaLinux 9 does not provide PHP 7.4 as an official Application Stream. Some legacy applications still require PHP 7.4 during migration. This repository explores how to retain the **AlmaLinux 8-maintained PHP 7.4 package and its security patch set** while adapting the RPM build to EL9 libraries and compiler behavior.

The aim is compatibility and migration testing, **not** to claim that PHP 7.4 is a current upstream-supported PHP version or that this EL9 port receives automatic security updates.

## Compatibility fixes made during this port

These changes address the specific build failures encountered while rebuilding `php-7.4.33-8.el9.src.rpm` with Mock on AlmaLinux 9 (`x86_64`):

| Component | Failure on AlmaLinux 9 | Porting adjustment |
| --- | --- | --- |
| **Net-SNMP / `ext/snmp`** | `usmDESPrivProtocol` is not declared by the available Net-SNMP headers. | Remove the unavailable DES protocol reference; do not silently substitute AES for DES. |
| **OpenSSL 3 / `ext/openssl`** | `RSA_SSLV23_PADDING` is not defined. | Conditionally register `OPENSSL_SSLV23_PADDING` only when that OpenSSL constant is available. |
| **GCC / Zend Engine** | LTO fails with `global register variable follows a function definition` in `Zend/zend_execute.c`. | Disable RPM link-time optimization using `%global _lto_cflags %{nil}` in the SPEC file. |
| **EL9 RPM build** | EL8 packaging assumptions and build dependencies differ from EL9. | Adapt the SPEC and rebuild in an AlmaLinux 9 Mock environment. |

These are **build-compatibility changes**, not newly developed fixes for PHP security vulnerabilities. In particular, removing a legacy SNMP DES reference does not restore DES support, and passing the OpenSSL build does not establish complete runtime compatibility with OpenSSL 3.

## Security patch provenance

The base package is the **AlmaLinux 8 `php:7.4` security-maintained RPM**, not the original unpatched PHP 7.4.33 tarball.

On October 21, 2026, AlmaLinux announced an updated `php:7.4` package, `php-7.4.33-8.module_el8.10.0+4321+cf5fd4bb`, addressing the PHP2Shell vulnerabilities **CVE-2026-109618** and **CVE-2026-109619**. That update was delivered through the normal AlmaLinux 8 repositories.

**Important:** The EL9 port must preserve and verify the security patches from the exact EL8 source RPM used for the rebuild. A matching PHP version or RPM release number is **not proof** that every security fix was retained. Future AlmaLinux 8 fixes must also be reviewed and imported manually; they do not automatically reach this EL9 repository.

## Build status

**Work in progress.** The previous SNMP and OpenSSL compilation blockers have been addressed in the ongoing build attempts. A subsequent build reached the linker and encountered the Zend/GCC LTO issue; disabling LTO was selected as the next fix.

**A successful complete RPM build, installation, extension tests, and runtime/security validation have not yet been confirmed.** This README does not represent the packages as production-ready.

## Building with Mock

The following commands assume that `php.spec` and the source files and patches referenced by it have been placed in a standard `~/rpmbuild` tree, and that `alma+epel-9-x86_64` is available as a Mock configuration.

```bash
rpmbuild -bs ~/rpmbuild/SPECS/php.spec

mock -r alma+epel-9-x86_64 --rebuild \
  ~/rpmbuild/SRPMS/php-7.4.33-8.el9.src.rpm
```

Build logs and artifacts:

```text
/var/lib/mock/alma+epel-9-x86_64/result/
```

After building, inspect the patch set and run installation, extension, integration, and security-regression tests before considering any deployment.

## Intended use and limitations

- Rebuild and compatibility testing for legacy PHP 7.4 applications on AlmaLinux 9.
- Short-term migration validation while moving to a supported PHP version.
- Research into maintaining downstream security backports across EL major versions.

This repository does **not** provide an official support contract, security-update SLA, warranty, or assurance that all extensions work. Prefer vendor-maintained, supported PHP versions for production wherever possible.

## References

- [AlmaLinux: PHP2Shell patched packages released (October 21, 2026)](https://almalinux.org/blog/2026-10-21-php2shell-cve-2026-109618-cve-2026-109619/)
- [AlmaLinux 8 PHP 7.4 security advisory ALSA-2026:62334](https://errata.almalinux.org/8/ALSA-2026-62334.html)
- [RHEL Application Streams Life Cycle](https://access.redhat.com/support/policy/updates/rhel-app-streams-life-cycle)

## Disclaimer

This is an independent, unofficial port. It is not affiliated with or endorsed by the PHP project, AlmaLinux OS Foundation, or Red Hat. All use is at your own risk.
