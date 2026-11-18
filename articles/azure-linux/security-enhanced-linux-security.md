---
title: Configure SELinux security in Azure Linux
description: Inspect SELinux enforcement and workload contexts, resolve policy denials, move workloads to enforcing mode, and port custom policy or AppArmor-confined workloads to Azure Linux.
author: pallakatos
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ai-usage: ai-assisted
ms.date: 10/13/2026
---

# Configure SELinux security in Azure Linux

Security-Enhanced Linux (SELinux) controls access through process and object
labels. Its policy determines which operations a process can perform on
labeled resources.

## Default SELinux mode and policy

Azure Linux 4 runs SELinux in enforcing mode with the targeted policy. The
`selinux-policy-targeted` and `policycoreutils` packages are installed. Keep
enforcement enabled; check labels and policy before changing the mode.

## Prerequisites

- An Azure Linux virtual machine (VM).
- A user account with `sudo` privileges.

## Verify SELinux mode, policy, and process contexts

Confirm the running mode, loaded policy, and workload context. An enforcing
system can also have unconfined services.

### Read the current state

Check the mode, policy, boot configuration, and installed tools that determine
how SELinux behaves.

#### Check the running mode and process contexts

Read the running mode and inspect process contexts:

```bash
getenforce
ps -eZ
```

Expected result from `getenforce` for the default configuration:

```output
Enforcing
```

In the process list, inspect the domain of the service you're investigating.
For example, `sshd_t` and `httpd_t` are service-specific domains.
`unconfined_service_t` doesn't provide the same service-specific confinement.
Don't infer workload protection from `Enforcing` alone.

#### Inspect the loaded policy

Read the policy and its properties:

```bash
sestatus
```

Expected policy and mode lines for the default configuration:

```output
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Max kernel policy version:      35
```

- `targeted` includes policies for selected services. Inspect each workload's
  context rather than assuming every process is confined.
- Multi-level security (MLS) support in the policy build isn't evidence that
  your workload uses MLS or container category isolation.
- The maximum kernel policy version reports what the kernel can load. It isn't
  a statement about the rules in the policy.

#### Check the configured mode and kernel command line

Read the configuration used at the next restart, and confirm that the kernel
command line isn't overriding it:

```bash
grep -E '^(SELINUX|SELINUXTYPE)=' /etc/selinux/config
cat /proc/cmdline
```

The configured values are:

```output
SELINUX=enforcing
SELINUXTYPE=targeted
```

Boot parameters such as `selinux=0` or `enforcing=0` can change startup
behavior. Inspect them alongside `/etc/selinux/config`.

#### Confirm installed SELinux tools

The image also includes `policycoreutils-python-utils`, which provides
`semanage`, `audit2allow`, and `audit2why`. Confirm the packages that are
present:

```bash
rpm -q selinux-policy-targeted policycoreutils policycoreutils-python-utils
```

Analysis tools such as `sesearch` and `seinfo` ship in `setools-console`, and
`udica`, which generates policies for containers, ships in `udica`. Both are
available from the Azure Linux repository but aren't installed by default.

## Resolve denials, port workloads, and change the SELinux mode

Required operations can be blocked by incorrect labels, nonstandard paths or
ports, or policy settings. Resolve those causes before considering a change to
enforcement mode.

### Investigate a denial

A denial identifies an operation the policy rejected, or would reject in
permissive mode. Check labels, paths, ports, and existing policy options before
adding a custom rule.

1. Collect recent denials:

   ```bash
   sudo ausearch -m AVC,USER_AVC -ts recent
   ```

   `<no matches>` means no matching records were found, not that no denial
   occurred. If necessary, temporarily disable `dontaudit` suppression:

   ```bash
   sudo semodule -DB
   ```

   Reproduce the failure, query the records again, and restore suppression with
   `sudo semodule -B`.

1. Translate the records into plain language:

   ```bash
   sudo ausearch -m AVC,USER_AVC -ts recent | audit2allow -w
   ```

   Read the source context, the target context, the object class, and the
   permission. Those four fields tell you which of the fixes below applies.

1. Check the labels involved. The dry run shows the current context next to the
   context the policy expects, without changing anything:

   ```bash
   ls -Z <path>
   sudo restorecon -nv <path>
   ```

   If the two differ, apply the expected label:

   ```bash
   sudo restorecon -Rv <path>
   ```

   For a permanent nonstandard location, add a file-context mapping first.
   See the example below.

1. Check whether a policy boolean covers the required behavior. Review its
   scope before enabling it; a boolean can grant more access than you need:

   ```bash
   getsebool -a | grep <keyword>
   sudo setsebool -P <boolean-name> on
   ```

   The `-P` flag writes the change to policy so it survives a restart.

1. If the service listens on a nonstandard port, label the port instead of
   changing the service's policy:

   ```bash
   sudo semanage port -l | grep <port-type>
   sudo semanage port -a -t <port-type> -p tcp <port-number>
   ```

> [!WARNING]
> `audit2allow -M` generates a policy module from whatever denials it's given.
> Applied without review, it can grant a compromised service exactly the access
> it was denied. Treat generated rules as a draft, read every line, and keep the
> module scoped to one problem.

### Example: label a nonstandard web-content directory

This example assumes the packaged Apache HTTP Server is already configured to
serve `/srv/web-content` at `http://127.0.0.1:8008/`. The service runs in
`httpd_t`, ordinary file permissions allow access, but requests fail with
HTTP 403 and an SELinux denial identifies `var_t` on the content.

Inspect the labels, record a persistent mapping for read-only web content, and
apply it:

```bash
ls -Zd /srv/web-content
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/web-content(/.*)?'
sudo restorecon -Rv /srv/web-content
matchpathcon -V /srv/web-content/index.html
```

The final command reports:

```output
/srv/web-content/index.html verified.
```

Request the file again and confirm enforcement remains enabled:

```bash
curl --fail --silent --show-error http://127.0.0.1:8008/index.html
getenforce
```

The request should return the file contents, and `getenforce` should return
`Enforcing`. This label permits serving content; it isn't a general-purpose
label for application-writable data. To undo the mapping, remove it with
`sudo semanage fcontext -d '/srv/web-content(/.*)?'`, then run
`sudo restorecon -Rv /srv/web-content`.

### Verify the fix

Repeat the original operation in enforcing mode. Check that it succeeds, then
inspect new denial records for that operation's time, process, and path:

```bash
sudo ausearch -m AVC,USER_AVC -ts recent
getenforce
```

`recent` includes the previous ten minutes, so the original denial can remain
in the results after a successful fix. Don't require the entire query to be
empty or treat an empty log as proof that the workload is correctly confined.

### Move a workload to enforcing mode

Use this procedure when you bring a workload from a system where SELinux was
permissive or disabled, or where the workload ran with a different policy.
Instead of making the whole VM permissive, make only the workload's domain
permissive while you collect denials. Other services stay enforced.

1. Identify the domain the workload's processes run in:

   ```bash
   ps -eZ | grep <process-name>
   ```

   The third field of the context, such as `httpd_t`, is the domain. If the
   domain is `unconfined_service_t`, no service-specific policy applies, and a
   permissive domain doesn't change its behavior.

1. Make that domain permissive:

   ```bash
   sudo semanage permissive -a <domain>
   sudo semanage permissive -l
   ```

   The domain is listed under `Customized Permissive Types`. The image also
   lists built-in permissive types; leave those unchanged.

1. Run the workload through its normal operations, including startup,
   representative requests, scheduled jobs, backups, and shutdown. Denials are
   logged but not enforced for this domain.

1. Review the denials and fix each one with a label, boolean, or port
   assignment, as described in [Investigate a denial](#investigate-a-denial):

   ```bash
   sudo ausearch -m AVC,USER_AVC -ts today -c <process-name> | audit2allow -w
   ```

1. If required access remains that no label, boolean, or port assignment
   covers, generate a local module from the remaining denials, then review it:

   ```bash
   sudo ausearch -m AVC,USER_AVC -ts today -c <process-name> \
     | audit2allow -M <module-name>
   cat <module-name>.te
   ```

   `audit2allow` writes `<module-name>.te` and `<module-name>.pp`. When a
   boolean already covers a denial, the `.te` file names it in a comment.
   Prefer the boolean. Remove any rule the workload doesn't need, then install
   the module:

   ```bash
   sudo semodule -i <module-name>.pp
   sudo semodule -l | grep <module-name>
   ```

   If you edit the `.te` file, rebuild the `.pp` file before installing it, as
   described in [Port a custom policy module](#port-a-custom-policy-module).

1. Remove the permissive exception and repeat the workload's operations:

   ```bash
   sudo semanage permissive -d <domain>
   ```

   Check for new denials as described in [Verify the fix](#verify-the-fix).

> [!WARNING]
> A permissive domain is a security exception for that service. Track it like
> any other exception, and remove it when the migration is complete.

### Port a custom policy module

A compiled module (`.pp`) depends on the types and attributes defined in the
policy it was built against. Rebuild custom modules from source on Azure Linux
instead of copying compiled modules from another system.

For a module with only a type enforcement file (`.te`), build and install it
with the tools in the image:

```bash
checkmodule -M -m -o <module-name>.mod <module-name>.te
semodule_package -o <module-name>.pp -m <module-name>.mod
sudo semodule -i <module-name>.pp
```

A module that also has file context (`.fc`) or interface (`.if`) files uses the
policy development makefile, which ships in `policycoreutils-devel`:

```bash
sudo dnf install -y policycoreutils-devel
make -f /usr/share/selinux/devel/Makefile <module-name>.pp
sudo semodule -i <module-name>.pp
```

After you install a module that defines file contexts, apply the labels to
existing files:

```bash
sudo restorecon -Rv <path>
```

If the module references a type or attribute that the Azure Linux policy
doesn't define, `semodule -i` fails with `Failed to resolve` messages. Find the
equivalent type in the Azure Linux policy, update the source, and rebuild.

To remove a module:

```bash
sudo semodule -r <module-name>
```

### Move a workload from a system that uses AppArmor

Azure Linux enforces mandatory access control with SELinux. AppArmor isn't
active, and its tools aren't available in the Azure Linux repository. No tool
converts AppArmor profiles into SELinux policy, so use the AppArmor profile
as a reference for what the workload needs, and confine the workload with
SELinux instead.

The two systems describe access differently:

| AppArmor | SELinux on Azure Linux |
| -------- | ---------------------- |
| A profile attached to an executable path | A domain, such as `httpd_t`, that a process enters when it runs a file with the matching entry-point label |
| File rules based on paths | Labels on files, assigned by file context rules (`semanage fcontext`) |
| Complain mode | A permissive domain |
| `aa-status` | `sestatus` and `semodule -l` |
| `aa-logprof` and `aa-genprof` | `audit2allow` and `sepolicy generate` |
| Abstractions and tunables | Policy interfaces and booleans (`getsebool -a`) |
| Network rules | Port labels (`semanage port`) |

> [!IMPORTANT]
> Do this work in a test or integration environment that runs the same
> Azure Linux image and the same workload configuration as production. Run
> the workload through all of its operations, and confirm that no denials
> remain, before you deploy the policy changes to production.

1. On the source system, list the profile that confines the workload and
   note the paths, capabilities, and network access it allows:

   ```bash
   sudo aa-status
   cat /etc/apparmor.d/<profile-name>
   ```

1. On Azure Linux, install and start the workload, then check the domain its
   processes run in:

   ```bash
   ps -eZ | grep <process-name>
   ```

   If the domain is service-specific, such as `httpd_t`, the Azure Linux
   policy already confines the workload. Continue with the next step. If the
   domain is `unconfined_service_t`, no policy applies to the workload. Go to
   step 4.

1. Map the workload's paths and ports to the existing policy. When the
   workload keeps data in a location the policy doesn't expect, label the
   location like the default location. For example, label `/srv/www` like
   `/var/www`:

   ```bash
   sudo semanage fcontext -a -e /var/www /srv/www
   sudo restorecon -Rv /srv/www
   ```

   Check for a boolean before you change policy, and assign nonstandard
   ports with `semanage port`, as described in
   [Investigate a denial](#investigate-a-denial). Then go to step 5.

1. If the workload has no domain, generate a policy for it from its
   executable. The `sepolicy generate` command ships in
   `policycoreutils-devel`:

   ```bash
   sudo dnf install -y policycoreutils-devel
   sepolicy generate --init -n <module-name> <path-to-executable>
   make -f /usr/share/selinux/devel/Makefile <module-name>.pp
   sudo semodule -i <module-name>.pp
   sudo restorecon -v <path-to-executable>
   sudo systemctl restart <service-name>
   ps -eZ | grep <process-name>
   ```

   The process now runs in the `<module-name>_t` domain. The generated
   `<module-name>.te` file contains a `permissive <module-name>_t;` line, so
   the new domain logs denials without enforcing them. For containers, use
   `udica` to generate a policy from the container's configuration instead.

1. Collect and fix denials as described in
   [Move a workload to enforcing mode](#move-a-workload-to-enforcing-mode),
   starting at the step that runs the workload's operations. For a domain
   that you generated, add the required rules to `<module-name>.te`. When no
   denials remain, remove the `permissive <module-name>_t;` line, rebuild the
   module, and install it again.

### Change the enforcement mode

These procedures apply to general-purpose Azure Linux VMs. For Azure Container
Linux (ACL) node pools on Azure Kubernetes Service (AKS), see
[Toggle SELinux mode for troubleshooting](./selinux-toggle-troubleshoot.md).

Permissive mode is a diagnostic exception when label and policy checks don't
isolate the problem. It stops enforcement for the entire VM and requires a
controlled diagnostic window.

```bash
sudo setenforce 0
getenforce
```

`Permissive` allows operations the policy would otherwise block and records
denials, subject to the policy's logging rules.

Return to enforcing as soon as you have the data:

```bash
sudo setenforce 1
getenforce
```

For an approved persistent change, set `SELINUX=enforcing` or
`SELINUX=permissive` in `/etc/selinux/config`, then restart and check
`getenforce`.

> [!CAUTION]
> Don't set `SELINUX=disabled` as a workaround. While SELinux is disabled, new
> and modified files aren't labeled, so re-enabling it later requires a full
> filesystem relabel that can add a long delay to the next start. Use
> permissive mode for diagnosis instead.

If you do need to relabel, for example after running with SELinux disabled,
schedule it during a maintenance window:

```bash
sudo fixfiles onboot
sudo reboot
```

## Troubleshoot SELinux enforcement and denials

### semanage: command not found

The command ships in `policycoreutils-python-utils`, which is included in the
image. If it was removed, reinstall it:

```bash
sudo dnf install -y policycoreutils-python-utils
```

### A service works in permissive mode and fails in enforcing mode

That result confirms SELinux is denying the operation. Follow
[Investigate a denial](#investigate-a-denial) and fix the label, boolean, or
port rather than leaving the system permissive.

### The system starts in the wrong mode

Check the kernel command line first:

```bash
cat /proc/cmdline
getenforce
```

A `selinux=0` or `enforcing=0` parameter overrides `/etc/selinux/config`.
Remove it from the boot configuration or from the image customization pipeline
that added it, then restart.

### Denials reappear after a restart or redeployment

Runtime label fixes are lost when a file is recreated, and `setsebool` without
`-P` is lost on restart. Record the intended state with `semanage fcontext` and
`setsebool -P`, then bake those steps into your image or configuration
management so a rebuilt VM starts in the same state.

## Related content

- [Azure Linux security overview](./security-overview.md)
- [Configure security logging and auditing](./logging-and-auditing.md)
- [Inspect kernel hardening](./kernel-hardening.md)
- [Toggle SELinux mode for troubleshooting](./selinux-toggle-troubleshoot.md)
- [SELinux notebook](https://github.com/SELinuxProject/selinux-notebook)
